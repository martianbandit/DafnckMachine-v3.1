# API Integration Documentation - DafnckMachine v3.1

## Document Metadata
- **Document Type**: API Integration Documentation
- **Version**: 3.1.0
- **Last Updated**: 2025-01-27
- **Author**: DafnckMachine Development Team
- **Status**: Active Implementation Guide

## Table of Contents
1. [API Integration Overview](#api-integration-overview)
2. [HTTP Client Configuration](#http-client-configuration)
3. [Data Fetching Strategies](#data-fetching-strategies)
4. [Error Handling Framework](#error-handling-framework)
5. [Loading States Management](#loading-states-management)
6. [Caching Strategies](#caching-strategies)
7. [Authentication Integration](#authentication-integration)
8. [Real-time Data Handling](#real-time-data-handling)
9. [API Testing Framework](#api-testing-framework)
10. [Performance Optimization](#performance-optimization)

## API Integration Overview

### Integration Architecture
The API integration layer provides a robust, scalable, and maintainable interface between the frontend application and backend services.

### Core Principles
- **Type Safety**: Full TypeScript integration with API contracts
- **Error Resilience**: Comprehensive error handling and recovery
- **Performance**: Optimized caching and request batching
- **Developer Experience**: Clear APIs and debugging tools
- **Scalability**: Modular architecture for easy extension

### Technology Stack
- **HTTP Client**: Axios with interceptors
- **State Management**: React Query for server state
- **Type Generation**: OpenAPI/Swagger code generation
- **WebSocket**: Socket.io for real-time features
- **Testing**: MSW (Mock Service Worker) for API mocking

## HTTP Client Configuration

### Axios Configuration
```typescript
// api/client.ts
import axios, { AxiosInstance, AxiosRequestConfig, AxiosResponse } from 'axios';
import { AuthService } from '../auth/AuthService';
import { ErrorHandler } from './ErrorHandler';

interface ApiClientConfig {
  baseURL: string;
  timeout: number;
  retryAttempts: number;
  retryDelay: number;
}

class ApiClient {
  private client: AxiosInstance;
  private config: ApiClientConfig;

  constructor(config: ApiClientConfig) {
    this.config = config;
    this.client = axios.create({
      baseURL: config.baseURL,
      timeout: config.timeout,
      headers: {
        'Content-Type': 'application/json',
      },
    });

    this.setupInterceptors();
  }

  private setupInterceptors(): void {
    // Request interceptor for authentication
    this.client.interceptors.request.use(
      (config) => {
        const token = AuthService.getToken();
        if (token) {
          config.headers.Authorization = `Bearer ${token}`;
        }
        return config;
      },
      (error) => Promise.reject(error)
    );

    // Response interceptor for error handling
    this.client.interceptors.response.use(
      (response) => response,
      async (error) => {
        const originalRequest = error.config;

        // Handle token refresh
        if (error.response?.status === 401 && !originalRequest._retry) {
          originalRequest._retry = true;
          
          try {
            await AuthService.refreshToken();
            const newToken = AuthService.getToken();
            originalRequest.headers.Authorization = `Bearer ${newToken}`;
            return this.client(originalRequest);
          } catch (refreshError) {
            AuthService.logout();
            window.location.href = '/login';
            return Promise.reject(refreshError);
          }
        }

        return Promise.reject(ErrorHandler.handleApiError(error));
      }
    );
  }

  // HTTP methods
  async get<T>(url: string, config?: AxiosRequestConfig): Promise<T> {
    const response = await this.client.get<T>(url, config);
    return response.data;
  }

  async post<T>(url: string, data?: any, config?: AxiosRequestConfig): Promise<T> {
    const response = await this.client.post<T>(url, data, config);
    return response.data;
  }

  async put<T>(url: string, data?: any, config?: AxiosRequestConfig): Promise<T> {
    const response = await this.client.put<T>(url, data, config);
    return response.data;
  }

  async patch<T>(url: string, data?: any, config?: AxiosRequestConfig): Promise<T> {
    const response = await this.client.patch<T>(url, data, config);
    return response.data;
  }

  async delete<T>(url: string, config?: AxiosRequestConfig): Promise<T> {
    const response = await this.client.delete<T>(url, config);
    return response.data;
  }
}

// Create and export configured client
export const apiClient = new ApiClient({
  baseURL: process.env.REACT_APP_API_BASE_URL || 'http://localhost:3001/api',
  timeout: 10000,
  retryAttempts: 3,
  retryDelay: 1000,
});
```

### Environment Configuration
```typescript
// config/api.ts
export const apiConfig = {
  development: {
    baseURL: 'http://localhost:3001/api',
    timeout: 10000,
    retryAttempts: 3,
  },
  staging: {
    baseURL: 'https://staging-api.dafnckmachine.com/api',
    timeout: 15000,
    retryAttempts: 2,
  },
  production: {
    baseURL: 'https://api.dafnckmachine.com/api',
    timeout: 20000,
    retryAttempts: 1,
  },
};

export const getApiConfig = () => {
  const env = process.env.NODE_ENV as keyof typeof apiConfig;
  return apiConfig[env] || apiConfig.development;
};
```

## Data Fetching Strategies

### React Query Integration
```typescript
// hooks/useApiQuery.ts
import { useQuery, useMutation, useQueryClient, UseQueryOptions } from '@tanstack/react-query';
import { apiClient } from '../api/client';

// Generic query hook
export function useApiQuery<T>(
  key: string[],
  fetcher: () => Promise<T>,
  options?: UseQueryOptions<T>
) {
  return useQuery({
    queryKey: key,
    queryFn: fetcher,
    staleTime: 5 * 60 * 1000, // 5 minutes
    cacheTime: 10 * 60 * 1000, // 10 minutes
    retry: 3,
    retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
    ...options,
  });
}

// Generic mutation hook
export function useApiMutation<T, V>(
  mutationFn: (variables: V) => Promise<T>,
  options?: {
    onSuccess?: (data: T, variables: V) => void;
    onError?: (error: Error, variables: V) => void;
    invalidateQueries?: string[][];
  }
) {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn,
    onSuccess: (data, variables) => {
      options?.onSuccess?.(data, variables);
      
      // Invalidate related queries
      options?.invalidateQueries?.forEach((queryKey) => {
        queryClient.invalidateQueries({ queryKey });
      });
    },
    onError: options?.onError,
  });
}
```

### Specific API Hooks
```typescript
// hooks/useUsers.ts
import { User, CreateUserRequest, UpdateUserRequest } from '../types/User';
import { useApiQuery, useApiMutation } from './useApiQuery';
import { apiClient } from '../api/client';

// Fetch users
export const useUsers = (params?: { page?: number; limit?: number; search?: string }) => {
  return useApiQuery(
    ['users', params],
    () => apiClient.get<{ users: User[]; total: number; page: number }>('/users', { params }),
    {
      keepPreviousData: true,
      staleTime: 2 * 60 * 1000, // 2 minutes for user data
    }
  );
};

// Fetch single user
export const useUser = (id: string) => {
  return useApiQuery(
    ['users', id],
    () => apiClient.get<User>(`/users/${id}`),
    {
      enabled: !!id,
    }
  );
};

// Create user mutation
export const useCreateUser = () => {
  return useApiMutation<User, CreateUserRequest>(
    (userData) => apiClient.post<User>('/users', userData),
    {
      onSuccess: () => {
        // Show success notification
        console.log('User created successfully');
      },
      invalidateQueries: [['users']],
    }
  );
};

// Update user mutation
export const useUpdateUser = () => {
  return useApiMutation<User, { id: string; data: UpdateUserRequest }>(
    ({ id, data }) => apiClient.put<User>(`/users/${id}`, data),
    {
      onSuccess: (data) => {
        console.log('User updated successfully');
      },
      invalidateQueries: [['users'], ['users', data.id]],
    }
  );
};

// Delete user mutation
export const useDeleteUser = () => {
  return useApiMutation<void, string>(
    (id) => apiClient.delete(`/users/${id}`),
    {
      onSuccess: () => {
        console.log('User deleted successfully');
      },
      invalidateQueries: [['users']],
    }
  );
};
```

### Infinite Queries for Pagination
```typescript
// hooks/useInfiniteUsers.ts
import { useInfiniteQuery } from '@tanstack/react-query';
import { apiClient } from '../api/client';

export const useInfiniteUsers = (search?: string) => {
  return useInfiniteQuery({
    queryKey: ['users', 'infinite', search],
    queryFn: ({ pageParam = 1 }) =>
      apiClient.get<{ users: User[]; hasMore: boolean; nextPage?: number }>('/users', {
        params: { page: pageParam, limit: 20, search },
      }),
    getNextPageParam: (lastPage) => lastPage.nextPage,
    staleTime: 5 * 60 * 1000,
  });
};
```

## Error Handling Framework

### Error Types and Classes
```typescript
// types/errors.ts
export enum ApiErrorType {
  NETWORK_ERROR = 'NETWORK_ERROR',
  VALIDATION_ERROR = 'VALIDATION_ERROR',
  AUTHENTICATION_ERROR = 'AUTHENTICATION_ERROR',
  AUTHORIZATION_ERROR = 'AUTHORIZATION_ERROR',
  NOT_FOUND_ERROR = 'NOT_FOUND_ERROR',
  SERVER_ERROR = 'SERVER_ERROR',
  TIMEOUT_ERROR = 'TIMEOUT_ERROR',
  UNKNOWN_ERROR = 'UNKNOWN_ERROR',
}

export interface ApiError {
  type: ApiErrorType;
  message: string;
  code?: string;
  details?: Record<string, any>;
  statusCode?: number;
}

export class ApiException extends Error {
  public readonly type: ApiErrorType;
  public readonly code?: string;
  public readonly details?: Record<string, any>;
  public readonly statusCode?: number;

  constructor(error: ApiError) {
    super(error.message);
    this.name = 'ApiException';
    this.type = error.type;
    this.code = error.code;
    this.details = error.details;
    this.statusCode = error.statusCode;
  }
}
```

### Error Handler Implementation
```typescript
// api/ErrorHandler.ts
import { AxiosError } from 'axios';
import { ApiError, ApiErrorType, ApiException } from '../types/errors';

export class ErrorHandler {
  static handleApiError(error: AxiosError): ApiException {
    if (error.response) {
      // Server responded with error status
      const { status, data } = error.response;
      
      switch (status) {
        case 400:
          return new ApiException({
            type: ApiErrorType.VALIDATION_ERROR,
            message: data?.message || 'Validation failed',
            code: data?.code,
            details: data?.details,
            statusCode: status,
          });
          
        case 401:
          return new ApiException({
            type: ApiErrorType.AUTHENTICATION_ERROR,
            message: 'Authentication required',
            statusCode: status,
          });
          
        case 403:
          return new ApiException({
            type: ApiErrorType.AUTHORIZATION_ERROR,
            message: 'Access denied',
            statusCode: status,
          });
          
        case 404:
          return new ApiException({
            type: ApiErrorType.NOT_FOUND_ERROR,
            message: 'Resource not found',
            statusCode: status,
          });
          
        case 500:
        default:
          return new ApiException({
            type: ApiErrorType.SERVER_ERROR,
            message: 'Internal server error',
            statusCode: status,
          });
      }
    } else if (error.request) {
      // Network error
      return new ApiException({
        type: ApiErrorType.NETWORK_ERROR,
        message: 'Network connection failed',
      });
    } else if (error.code === 'ECONNABORTED') {
      // Timeout error
      return new ApiException({
        type: ApiErrorType.TIMEOUT_ERROR,
        message: 'Request timeout',
      });
    } else {
      // Unknown error
      return new ApiException({
        type: ApiErrorType.UNKNOWN_ERROR,
        message: error.message || 'An unknown error occurred',
      });
    }
  }

  static getErrorMessage(error: ApiException): string {
    switch (error.type) {
      case ApiErrorType.NETWORK_ERROR:
        return 'Please check your internet connection and try again.';
      case ApiErrorType.VALIDATION_ERROR:
        return error.details?.message || 'Please check your input and try again.';
      case ApiErrorType.AUTHENTICATION_ERROR:
        return 'Please log in to continue.';
      case ApiErrorType.AUTHORIZATION_ERROR:
        return 'You do not have permission to perform this action.';
      case ApiErrorType.NOT_FOUND_ERROR:
        return 'The requested resource was not found.';
      case ApiErrorType.TIMEOUT_ERROR:
        return 'The request took too long. Please try again.';
      case ApiErrorType.SERVER_ERROR:
        return 'A server error occurred. Please try again later.';
      default:
        return 'An unexpected error occurred. Please try again.';
    }
  }
}
```

### Error Boundary Component
```typescript
// components/ErrorBoundary.tsx
import React, { Component, ErrorInfo, ReactNode } from 'react';
import { ApiException } from '../types/errors';
import { ErrorHandler } from '../api/ErrorHandler';

interface Props {
  children: ReactNode;
  fallback?: (error: Error) => ReactNode;
}

interface State {
  hasError: boolean;
  error?: Error;
}

export class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
    
    // Log error to monitoring service
    if (process.env.NODE_ENV === 'production') {
      // Send to error tracking service
    }
  }

  render() {
    if (this.state.hasError) {
      if (this.props.fallback) {
        return this.props.fallback(this.state.error!);
      }

      const errorMessage = this.state.error instanceof ApiException
        ? ErrorHandler.getErrorMessage(this.state.error)
        : 'An unexpected error occurred.';

      return (
        <div className="error-boundary">
          <h2>Something went wrong</h2>
          <p>{errorMessage}</p>
          <button onClick={() => this.setState({ hasError: false })}>
            Try again
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}
```

## Loading States Management

### Loading State Hook
```typescript
// hooks/useLoadingState.ts
import { useState, useCallback } from 'react';

interface LoadingState {
  isLoading: boolean;
  error: Error | null;
  data: any;
}

export const useLoadingState = <T>() => {
  const [state, setState] = useState<LoadingState>({
    isLoading: false,
    error: null,
    data: null,
  });

  const execute = useCallback(async (asyncFunction: () => Promise<T>) => {
    setState({ isLoading: true, error: null, data: null });
    
    try {
      const result = await asyncFunction();
      setState({ isLoading: false, error: null, data: result });
      return result;
    } catch (error) {
      setState({ isLoading: false, error: error as Error, data: null });
      throw error;
    }
  }, []);

  const reset = useCallback(() => {
    setState({ isLoading: false, error: null, data: null });
  }, []);

  return {
    ...state,
    execute,
    reset,
  };
};
```

### Loading Components
```typescript
// components/LoadingSpinner.tsx
import React from 'react';
import styled, { keyframes } from 'styled-components';

const spin = keyframes`
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
`;

const Spinner = styled.div<{ size?: 'sm' | 'md' | 'lg' }>`
  border: 2px solid ${({ theme }) => theme.colors.neutral[200]};
  border-top: 2px solid ${({ theme }) => theme.colors.primary[500]};
  border-radius: 50%;
  width: ${({ size = 'md' }) => 
    size === 'sm' ? '16px' : size === 'lg' ? '48px' : '32px'};
  height: ${({ size = 'md' }) => 
    size === 'sm' ? '16px' : size === 'lg' ? '48px' : '32px'};
  animation: ${spin} 1s linear infinite;
`;

const Container = styled.div<{ fullScreen?: boolean }>`
  display: flex;
  justify-content: center;
  align-items: center;
  ${({ fullScreen }) => fullScreen && `
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: rgba(255, 255, 255, 0.8);
    z-index: 9999;
  `}
`;

interface LoadingSpinnerProps {
  size?: 'sm' | 'md' | 'lg';
  fullScreen?: boolean;
}

export const LoadingSpinner: React.FC<LoadingSpinnerProps> = ({ 
  size = 'md', 
  fullScreen = false 
}) => (
  <Container fullScreen={fullScreen}>
    <Spinner size={size} />
  </Container>
);

// Skeleton loading component
const SkeletonBox = styled.div<{ width?: string; height?: string }>`
  background: linear-gradient(
    90deg,
    ${({ theme }) => theme.colors.neutral[200]} 25%,
    ${({ theme }) => theme.colors.neutral[100]} 50%,
    ${({ theme }) => theme.colors.neutral[200]} 75%
  );
  background-size: 200% 100%;
  animation: ${keyframes`
    0% { background-position: 200% 0; }
    100% { background-position: -200% 0; }
  `} 1.5s ease-in-out infinite;
  border-radius: 4px;
  width: ${({ width = '100%' }) => width};
  height: ${({ height = '20px' }) => height};
`;

export const Skeleton: React.FC<{ width?: string; height?: string }> = ({ 
  width, 
  height 
}) => <SkeletonBox width={width} height={height} />;
```

### Suspense Integration
```typescript
// components/SuspenseWrapper.tsx
import React, { Suspense } from 'react';
import { LoadingSpinner } from './LoadingSpinner';
import { ErrorBoundary } from './ErrorBoundary';

interface SuspenseWrapperProps {
  children: React.ReactNode;
  fallback?: React.ReactNode;
  errorFallback?: (error: Error) => React.ReactNode;
}

export const SuspenseWrapper: React.FC<SuspenseWrapperProps> = ({
  children,
  fallback = <LoadingSpinner />,
  errorFallback,
}) => (
  <ErrorBoundary fallback={errorFallback}>
    <Suspense fallback={fallback}>
      {children}
    </Suspense>
  </ErrorBoundary>
);
```

## Caching Strategies

### React Query Cache Configuration
```typescript
// config/queryClient.ts
import { QueryClient } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      // Global defaults
      staleTime: 5 * 60 * 1000, // 5 minutes
      cacheTime: 10 * 60 * 1000, // 10 minutes
      retry: 3,
      retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
      refetchOnWindowFocus: false,
      refetchOnReconnect: true,
    },
    mutations: {
      retry: 1,
    },
  },
});
```

### Cache Invalidation Strategies
```typescript
// utils/cacheUtils.ts
import { QueryClient } from '@tanstack/react-query';

export class CacheManager {
  constructor(private queryClient: QueryClient) {}

  // Invalidate all queries for a resource
  invalidateResource(resource: string): void {
    this.queryClient.invalidateQueries({ queryKey: [resource] });
  }

  // Invalidate specific query
  invalidateQuery(queryKey: string[]): void {
    this.queryClient.invalidateQueries({ queryKey });
  }

  // Remove query from cache
  removeQuery(queryKey: string[]): void {
    this.queryClient.removeQueries({ queryKey });
  }

  // Update query data optimistically
  updateQueryData<T>(queryKey: string[], updater: (old: T | undefined) => T): void {
    this.queryClient.setQueryData(queryKey, updater);
  }

  // Prefetch data
  async prefetchQuery<T>(
    queryKey: string[],
    queryFn: () => Promise<T>,
    staleTime?: number
  ): Promise<void> {
    await this.queryClient.prefetchQuery({
      queryKey,
      queryFn,
      staleTime,
    });
  }

  // Clear all cache
  clearCache(): void {
    this.queryClient.clear();
  }
}

export const cacheManager = new CacheManager(queryClient);
```

### Optimistic Updates
```typescript
// hooks/useOptimisticUpdate.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';

export const useOptimisticUserUpdate = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ id, data }: { id: string; data: Partial<User> }) =>
      apiClient.put<User>(`/users/${id}`, data),
    
    onMutate: async ({ id, data }) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: ['users', id] });

      // Snapshot previous value
      const previousUser = queryClient.getQueryData<User>(['users', id]);

      // Optimistically update
      queryClient.setQueryData<User>(['users', id], (old) => ({
        ...old!,
        ...data,
      }));

      return { previousUser };
    },
    
    onError: (err, { id }, context) => {
      // Rollback on error
      if (context?.previousUser) {
        queryClient.setQueryData(['users', id], context.previousUser);
      }
    },
    
    onSettled: (data, error, { id }) => {
      // Refetch to ensure consistency
      queryClient.invalidateQueries({ queryKey: ['users', id] });
    },
  });
};
```

## Authentication Integration

### Auth Context and Hooks
```typescript
// auth/AuthContext.tsx
import React, { createContext, useContext, useEffect, useState } from 'react';
import { User } from '../types/User';
import { AuthService } from './AuthService';

interface AuthContextType {
  user: User | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  refreshToken: () => Promise<void>;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export const AuthProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    const initAuth = async () => {
      try {
        const token = AuthService.getToken();
        if (token) {
          const userData = await AuthService.getCurrentUser();
          setUser(userData);
        }
      } catch (error) {
        console.error('Auth initialization failed:', error);
        AuthService.logout();
      } finally {
        setIsLoading(false);
      }
    };

    initAuth();
  }, []);

  const login = async (email: string, password: string) => {
    const { user: userData, token } = await AuthService.login(email, password);
    AuthService.setToken(token);
    setUser(userData);
  };

  const logout = () => {
    AuthService.logout();
    setUser(null);
  };

  const refreshToken = async () => {
    await AuthService.refreshToken();
  };

  return (
    <AuthContext.Provider
      value={{
        user,
        isAuthenticated: !!user,
        isLoading,
        login,
        logout,
        refreshToken,
      }}
    >
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
};
```

### Protected Route Component
```typescript
// components/ProtectedRoute.tsx
import React from 'react';
import { Navigate, useLocation } from 'react-router-dom';
import { useAuth } from '../auth/AuthContext';
import { LoadingSpinner } from './LoadingSpinner';

interface ProtectedRouteProps {
  children: React.ReactNode;
  requiredRole?: string;
}

export const ProtectedRoute: React.FC<ProtectedRouteProps> = ({
  children,
  requiredRole,
}) => {
  const { isAuthenticated, isLoading, user } = useAuth();
  const location = useLocation();

  if (isLoading) {
    return <LoadingSpinner fullScreen />;
  }

  if (!isAuthenticated) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  if (requiredRole && user?.role !== requiredRole) {
    return <Navigate to="/unauthorized" replace />;
  }

  return <>{children}</>;
};
```

## Real-time Data Handling

### WebSocket Integration
```typescript
// websocket/WebSocketClient.ts
import { io, Socket } from 'socket.io-client';
import { AuthService } from '../auth/AuthService';

export class WebSocketClient {
  private socket: Socket | null = null;
  private reconnectAttempts = 0;
  private maxReconnectAttempts = 5;

  connect(): void {
    const token = AuthService.getToken();
    
    this.socket = io(process.env.REACT_APP_WS_URL || 'ws://localhost:3001', {
      auth: {
        token,
      },
      transports: ['websocket'],
    });

    this.setupEventListeners();
  }

  private setupEventListeners(): void {
    if (!this.socket) return;

    this.socket.on('connect', () => {
      console.log('WebSocket connected');
      this.reconnectAttempts = 0;
    });

    this.socket.on('disconnect', (reason) => {
      console.log('WebSocket disconnected:', reason);
      
      if (reason === 'io server disconnect') {
        // Server disconnected, try to reconnect
        this.handleReconnect();
      }
    });

    this.socket.on('connect_error', (error) => {
      console.error('WebSocket connection error:', error);
      this.handleReconnect();
    });
  }

  private handleReconnect(): void {
    if (this.reconnectAttempts < this.maxReconnectAttempts) {
      this.reconnectAttempts++;
      setTimeout(() => {
        this.socket?.connect();
      }, Math.pow(2, this.reconnectAttempts) * 1000);
    }
  }

  subscribe(event: string, callback: (data: any) => void): void {
    this.socket?.on(event, callback);
  }

  unsubscribe(event: string, callback?: (data: any) => void): void {
    if (callback) {
      this.socket?.off(event, callback);
    } else {
      this.socket?.removeAllListeners(event);
    }
  }

  emit(event: string, data: any): void {
    this.socket?.emit(event, data);
  }

  disconnect(): void {
    this.socket?.disconnect();
    this.socket = null;
  }
}

export const wsClient = new WebSocketClient();
```

### Real-time Data Hooks
```typescript
// hooks/useWebSocket.ts
import { useEffect, useRef } from 'react';
import { useQueryClient } from '@tanstack/react-query';
import { wsClient } from '../websocket/WebSocketClient';

export const useWebSocket = () => {
  const queryClient = useQueryClient();
  const isConnected = useRef(false);

  useEffect(() => {
    if (!isConnected.current) {
      wsClient.connect();
      isConnected.current = true;
    }

    // Handle real-time updates
    const handleUserUpdate = (data: { userId: string; user: User }) => {
      queryClient.setQueryData(['users', data.userId], data.user);
      queryClient.invalidateQueries({ queryKey: ['users'] });
    };

    const handleNotification = (notification: Notification) => {
      queryClient.setQueryData(
        ['notifications'],
        (old: Notification[] = []) => [notification, ...old]
      );
    };

    wsClient.subscribe('user:updated', handleUserUpdate);
    wsClient.subscribe('notification:new', handleNotification);

    return () => {
      wsClient.unsubscribe('user:updated', handleUserUpdate);
      wsClient.unsubscribe('notification:new', handleNotification);
    };
  }, [queryClient]);

  return {
    emit: wsClient.emit.bind(wsClient),
    subscribe: wsClient.subscribe.bind(wsClient),
    unsubscribe: wsClient.unsubscribe.bind(wsClient),
  };
};
```

## API Testing Framework

### MSW Setup for API Mocking
```typescript
// mocks/handlers.ts
import { rest } from 'msw';
import { User } from '../types/User';

const users: User[] = [
  { id: '1', name: 'John Doe', email: 'john@example.com', role: 'admin' },
  { id: '2', name: 'Jane Smith', email: 'jane@example.com', role: 'user' },
];

export const handlers = [
  // Get users
  rest.get('/api/users', (req, res, ctx) => {
    const page = Number(req.url.searchParams.get('page')) || 1;
    const limit = Number(req.url.searchParams.get('limit')) || 10;
    const search = req.url.searchParams.get('search');

    let filteredUsers = users;
    if (search) {
      filteredUsers = users.filter(user =>
        user.name.toLowerCase().includes(search.toLowerCase()) ||
        user.email.toLowerCase().includes(search.toLowerCase())
      );
    }

    const start = (page - 1) * limit;
    const end = start + limit;
    const paginatedUsers = filteredUsers.slice(start, end);

    return res(
      ctx.status(200),
      ctx.json({
        users: paginatedUsers,
        total: filteredUsers.length,
        page,
        hasMore: end < filteredUsers.length,
      })
    );
  }),

  // Get single user
  rest.get('/api/users/:id', (req, res, ctx) => {
    const { id } = req.params;
    const user = users.find(u => u.id === id);

    if (!user) {
      return res(ctx.status(404), ctx.json({ message: 'User not found' }));
    }

    return res(ctx.status(200), ctx.json(user));
  }),

  // Create user
  rest.post('/api/users', async (req, res, ctx) => {
    const userData = await req.json();
    const newUser: User = {
      id: String(users.length + 1),
      ...userData,
    };
    users.push(newUser);

    return res(ctx.status(201), ctx.json(newUser));
  }),

  // Update user
  rest.put('/api/users/:id', async (req, res, ctx) => {
    const { id } = req.params;
    const userData = await req.json();
    const userIndex = users.findIndex(u => u.id === id);

    if (userIndex === -1) {
      return res(ctx.status(404), ctx.json({ message: 'User not found' }));
    }

    users[userIndex] = { ...users[userIndex], ...userData };
    return res(ctx.status(200), ctx.json(users[userIndex]));
  }),

  // Delete user
  rest.delete('/api/users/:id', (req, res, ctx) => {
    const { id } = req.params;
    const userIndex = users.findIndex(u => u.id === id);

    if (userIndex === -1) {
      return res(ctx.status(404), ctx.json({ message: 'User not found' }));
    }

    users.splice(userIndex, 1);
    return res(ctx.status(204));
  }),

  // Error simulation
  rest.get('/api/error', (req, res, ctx) => {
    return res(ctx.status(500), ctx.json({ message: 'Internal server error' }));
  }),
];
```

### Test Utilities
```typescript
// test-utils/api-test-utils.ts
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { render, RenderOptions } from '@testing-library/react';
import { ReactElement } from 'react';

// Create a test query client
const createTestQueryClient = () =>
  new QueryClient({
    defaultOptions: {
      queries: {
        retry: false,
        cacheTime: 0,
      },
    },
  });

// Custom render with providers
export const renderWithProviders = (
  ui: ReactElement,
  options?: Omit<RenderOptions, 'wrapper'>
) => {
  const testQueryClient = createTestQueryClient();

  const Wrapper = ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={testQueryClient}>
      {children}
    </QueryClientProvider>
  );

  return render(ui, { wrapper: Wrapper, ...options });
};

// Wait for API calls to complete
export const waitForApiCalls = () => new Promise(resolve => setTimeout(resolve, 0));
```

## Performance Optimization

### Request Batching
```typescript
// utils/requestBatcher.ts
interface BatchRequest {
  id: string;
  resolve: (data: any) => void;
  reject: (error: any) => void;
}

export class RequestBatcher {
  private batches = new Map<string, BatchRequest[]>();
  private timeouts = new Map<string, NodeJS.Timeout>();

  batch<T>(
    key: string,
    request: () => Promise<T>,
    delay = 50
  ): Promise<T> {
    return new Promise((resolve, reject) => {
      const id = Math.random().toString(36);
      const batchRequest: BatchRequest = { id, resolve, reject };

      if (!this.batches.has(key)) {
        this.batches.set(key, []);
      }

      this.batches.get(key)!.push(batchRequest);

      // Clear existing timeout
      if (this.timeouts.has(key)) {
        clearTimeout(this.timeouts.get(key)!);
      }

      // Set new timeout
      const timeout = setTimeout(async () => {
        const requests = this.batches.get(key) || [];
        this.batches.delete(key);
        this.timeouts.delete(key);

        try {
          const result = await request();
          requests.forEach(req => req.resolve(result));
        } catch (error) {
          requests.forEach(req => req.reject(error));
        }
      }, delay);

      this.timeouts.set(key, timeout);
    });
  }
}

export const requestBatcher = new RequestBatcher();
```

### Request Deduplication
```typescript
// utils/requestDeduplicator.ts
export class RequestDeduplicator {
  private pendingRequests = new Map<string, Promise<any>>();

  async dedupe<T>(key: string, request: () => Promise<T>): Promise<T> {
    if (this.pendingRequests.has(key)) {
      return this.pendingRequests.get(key);
    }

    const promise = request().finally(() => {
      this.pendingRequests.delete(key);
    });

    this.pendingRequests.set(key, promise);
    return promise;
  }
}

export const requestDeduplicator = new RequestDeduplicator();
```

### Performance Monitoring
```typescript
// utils/performanceMonitor.ts
export class PerformanceMonitor {
  static measureApiCall<T>(
    name: string,
    request: () => Promise<T>
  ): Promise<T> {
    const startTime = performance.now();
    
    return request().finally(() => {
      const endTime = performance.now();
      const duration = endTime - startTime;
      
      console.log(`API call ${name} took ${duration.toFixed(2)}ms`);
      
      // Send to analytics service
      if (process.env.NODE_ENV === 'production') {
        // Analytics.track('api_call_duration', { name, duration });
      }
    });
  }

  static measureQueryPerformance() {
    // Monitor React Query performance
    return {
      onSuccess: (data: any, query: any) => {
        console.log(`Query ${query.queryKey.join(':')} succeeded`);
      },
      onError: (error: any, query: any) => {
        console.error(`Query ${query.queryKey.join(':')} failed:`, error);
      },
    };
  }
}
```

---

**Document Status**: Active Implementation Guide  
**Last Review**: 2025-01-27  
**Next Review**: 2025-02-27  
**Maintained By**: DafnckMachine Development Team 