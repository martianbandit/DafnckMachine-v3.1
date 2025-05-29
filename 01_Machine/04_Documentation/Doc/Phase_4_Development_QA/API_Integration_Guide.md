# API Integration Guide - DafnckMachine v3.1

## Document Metadata
- **Document Type**: API Integration Guide
- **Version**: 3.1.0
- **Last Updated**: 2025-01-27
- **Owner**: Frontend Development Team
- **Status**: Active Development

## Overview

This document provides comprehensive guidelines for API integration in DafnckMachine v3.1, covering HTTP client configuration, authentication handling, error management, caching strategies, real-time communication, and testing approaches.

## API Integration Architecture

### Integration Layer Structure
```
API Integration Layer
├── HTTP Client (Axios)
│   ├── Request Interceptors
│   ├── Response Interceptors
│   ├── Error Handling
│   └── Authentication
├── API Services
│   ├── Authentication API
│   ├── User Management API
│   ├── Data Management API
│   └── File Upload API
├── State Management
│   ├── React Query (Server State)
│   ├── Optimistic Updates
│   ├── Background Sync
│   └── Cache Management
└── Real-time Communication
    ├── WebSocket Connection
    ├── Server-Sent Events
    ├── Push Notifications
    └── Offline Support
```

## HTTP Client Configuration

### Axios Setup and Configuration

```typescript
// src/lib/api/client.ts
import axios, { AxiosInstance, AxiosRequestConfig, AxiosResponse, AxiosError } from 'axios';
import { useAuthStore } from '@/stores/authStore';
import { useAppStore } from '@/stores/appStore';

export interface ApiResponse<T = any> {
  data: T;
  message?: string;
  success: boolean;
  errors?: Record<string, string[]>;
  meta?: {
    pagination?: {
      page: number;
      limit: number;
      total: number;
      totalPages: number;
    };
    timestamp: string;
    requestId: string;
  };
}

export interface ApiError {
  message: string;
  status: number;
  code?: string;
  errors?: Record<string, string[]>;
  requestId?: string;
}

class ApiClient {
  private client: AxiosInstance;
  private baseURL: string;

  constructor(baseURL: string) {
    this.baseURL = baseURL;
    this.client = axios.create({
      baseURL,
      timeout: 30000,
      headers: {
        'Content-Type': 'application/json',
        'Accept': 'application/json',
      },
    });

    this.setupInterceptors();
  }

  private setupInterceptors() {
    // Request interceptor
    this.client.interceptors.request.use(
      (config) => {
        // Add authentication token
        const token = useAuthStore.getState().token;
        if (token) {
          config.headers.Authorization = `Bearer ${token}`;
        }

        // Add request ID for tracking
        config.headers['X-Request-ID'] = this.generateRequestId();

        // Add timestamp
        config.headers['X-Timestamp'] = new Date().toISOString();

        // Log request in development
        if (process.env.NODE_ENV === 'development') {
          console.log(`🚀 API Request: ${config.method?.toUpperCase()} ${config.url}`, {
            headers: config.headers,
            data: config.data,
            params: config.params,
          });
        }

        return config;
      },
      (error) => {
        console.error('Request interceptor error:', error);
        return Promise.reject(error);
      }
    );

    // Response interceptor
    this.client.interceptors.response.use(
      (response: AxiosResponse<ApiResponse>) => {
        // Log response in development
        if (process.env.NODE_ENV === 'development') {
          console.log(`✅ API Response: ${response.config.method?.toUpperCase()} ${response.config.url}`, {
            status: response.status,
            data: response.data,
          });
        }

        return response;
      },
      (error: AxiosError<ApiResponse>) => {
        return this.handleResponseError(error);
      }
    );
  }

  private handleResponseError(error: AxiosError<ApiResponse>): Promise<never> {
    const { response, request, message } = error;

    // Network error
    if (!response) {
      const networkError: ApiError = {
        message: 'Network error. Please check your connection.',
        status: 0,
        code: 'NETWORK_ERROR',
      };
      
      useAppStore.getState().addNotification({
        type: 'error',
        title: 'Network Error',
        message: networkError.message,
      });

      return Promise.reject(networkError);
    }

    // HTTP error response
    const { status, data } = response;
    const apiError: ApiError = {
      message: data?.message || this.getDefaultErrorMessage(status),
      status,
      code: data?.code,
      errors: data?.errors,
      requestId: response.headers['x-request-id'],
    };

    // Handle specific error cases
    switch (status) {
      case 401:
        this.handleUnauthorizedError();
        break;
      case 403:
        this.handleForbiddenError();
        break;
      case 422:
        this.handleValidationError(apiError);
        break;
      case 429:
        this.handleRateLimitError();
        break;
      case 500:
      case 502:
      case 503:
      case 504:
        this.handleServerError(apiError);
        break;
    }

    // Log error in development
    if (process.env.NODE_ENV === 'development') {
      console.error(`❌ API Error: ${error.config?.method?.toUpperCase()} ${error.config?.url}`, {
        status,
        message: apiError.message,
        errors: apiError.errors,
        requestId: apiError.requestId,
      });
    }

    return Promise.reject(apiError);
  }

  private handleUnauthorizedError() {
    const authStore = useAuthStore.getState();
    
    // Only logout if user was previously authenticated
    if (authStore.isAuthenticated) {
      authStore.logout();
      useAppStore.getState().addNotification({
        type: 'error',
        title: 'Session Expired',
        message: 'Your session has expired. Please log in again.',
      });
      
      // Redirect to login page
      window.location.href = '/auth/login';
    }
  }

  private handleForbiddenError() {
    useAppStore.getState().addNotification({
      type: 'error',
      title: 'Access Denied',
      message: 'You do not have permission to perform this action.',
    });
  }

  private handleValidationError(error: ApiError) {
    if (error.errors) {
      // Handle field-specific validation errors
      Object.entries(error.errors).forEach(([field, messages]) => {
        messages.forEach(message => {
          useAppStore.getState().addNotification({
            type: 'error',
            title: `Validation Error: ${field}`,
            message,
          });
        });
      });
    }
  }

  private handleRateLimitError() {
    useAppStore.getState().addNotification({
      type: 'warning',
      title: 'Rate Limit Exceeded',
      message: 'Too many requests. Please wait a moment before trying again.',
    });
  }

  private handleServerError(error: ApiError) {
    useAppStore.getState().addNotification({
      type: 'error',
      title: 'Server Error',
      message: 'Something went wrong on our end. Please try again later.',
    });

    // Report to error tracking service
    if (window.Sentry) {
      window.Sentry.captureException(new Error(error.message), {
        tags: {
          component: 'api-client',
          status: error.status,
        },
        extra: {
          requestId: error.requestId,
          errors: error.errors,
        },
      });
    }
  }

  private getDefaultErrorMessage(status: number): string {
    switch (status) {
      case 400: return 'Bad request. Please check your input.';
      case 401: return 'Authentication required.';
      case 403: return 'Access denied.';
      case 404: return 'Resource not found.';
      case 422: return 'Validation failed.';
      case 429: return 'Too many requests.';
      case 500: return 'Internal server error.';
      case 502: return 'Bad gateway.';
      case 503: return 'Service unavailable.';
      case 504: return 'Gateway timeout.';
      default: return 'An unexpected error occurred.';
    }
  }

  private generateRequestId(): string {
    return `req_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }

  // HTTP methods
  async get<T = any>(url: string, config?: AxiosRequestConfig): Promise<ApiResponse<T>> {
    const response = await this.client.get<ApiResponse<T>>(url, config);
    return response.data;
  }

  async post<T = any>(url: string, data?: any, config?: AxiosRequestConfig): Promise<ApiResponse<T>> {
    const response = await this.client.post<ApiResponse<T>>(url, data, config);
    return response.data;
  }

  async put<T = any>(url: string, data?: any, config?: AxiosRequestConfig): Promise<ApiResponse<T>> {
    const response = await this.client.put<ApiResponse<T>>(url, data, config);
    return response.data;
  }

  async patch<T = any>(url: string, data?: any, config?: AxiosRequestConfig): Promise<ApiResponse<T>> {
    const response = await this.client.patch<ApiResponse<T>>(url, data, config);
    return response.data;
  }

  async delete<T = any>(url: string, config?: AxiosRequestConfig): Promise<ApiResponse<T>> {
    const response = await this.client.delete<ApiResponse<T>>(url, config);
    return response.data;
  }

  // File upload with progress
  async upload<T = any>(
    url: string,
    file: File,
    onProgress?: (progress: number) => void
  ): Promise<ApiResponse<T>> {
    const formData = new FormData();
    formData.append('file', file);

    const response = await this.client.post<ApiResponse<T>>(url, formData, {
      headers: {
        'Content-Type': 'multipart/form-data',
      },
      onUploadProgress: (progressEvent) => {
        if (onProgress && progressEvent.total) {
          const progress = Math.round((progressEvent.loaded * 100) / progressEvent.total);
          onProgress(progress);
        }
      },
    });

    return response.data;
  }

  // Set authentication token
  setToken(token: string) {
    this.client.defaults.headers.common['Authorization'] = `Bearer ${token}`;
  }

  // Clear authentication token
  clearToken() {
    delete this.client.defaults.headers.common['Authorization'];
  }

  // Update base URL
  setBaseURL(baseURL: string) {
    this.baseURL = baseURL;
    this.client.defaults.baseURL = baseURL;
  }
}

// Create and export API client instance
export const apiClient = new ApiClient(
  process.env.REACT_APP_API_URL || 'http://localhost:3001/api'
);
```

## API Service Layer

### Authentication API Service

```typescript
// src/api/auth.ts
import { apiClient, ApiResponse } from '@/lib/api/client';

export interface LoginCredentials {
  email: string;
  password: string;
  rememberMe?: boolean;
}

export interface RegisterData {
  firstName: string;
  lastName: string;
  email: string;
  password: string;
  confirmPassword: string;
  acceptTerms: boolean;
}

export interface AuthResponse {
  user: User;
  token: string;
  refreshToken: string;
  expiresIn: number;
}

export interface RefreshTokenResponse {
  token: string;
  expiresIn: number;
}

export interface ResetPasswordData {
  token: string;
  password: string;
  confirmPassword: string;
}

class AuthApi {
  private readonly basePath = '/auth';

  async login(credentials: LoginCredentials): Promise<AuthResponse> {
    const response = await apiClient.post<AuthResponse>(
      `${this.basePath}/login`,
      credentials
    );
    return response.data;
  }

  async register(data: RegisterData): Promise<AuthResponse> {
    const response = await apiClient.post<AuthResponse>(
      `${this.basePath}/register`,
      data
    );
    return response.data;
  }

  async logout(): Promise<void> {
    await apiClient.post(`${this.basePath}/logout`);
  }

  async refreshToken(refreshToken: string): Promise<RefreshTokenResponse> {
    const response = await apiClient.post<RefreshTokenResponse>(
      `${this.basePath}/refresh`,
      { refreshToken }
    );
    return response.data;
  }

  async forgotPassword(email: string): Promise<void> {
    await apiClient.post(`${this.basePath}/forgot-password`, { email });
  }

  async resetPassword(data: ResetPasswordData): Promise<void> {
    await apiClient.post(`${this.basePath}/reset-password`, data);
  }

  async verifyEmail(token: string): Promise<void> {
    await apiClient.post(`${this.basePath}/verify-email`, { token });
  }

  async resendVerification(email: string): Promise<void> {
    await apiClient.post(`${this.basePath}/resend-verification`, { email });
  }

  async changePassword(data: {
    currentPassword: string;
    newPassword: string;
    confirmPassword: string;
  }): Promise<void> {
    await apiClient.post(`${this.basePath}/change-password`, data);
  }

  // Set token in client
  setToken(token: string) {
    apiClient.setToken(token);
  }

  // Clear token from client
  clearToken() {
    apiClient.clearToken();
  }
}

export const authApi = new AuthApi();
```

### User Management API Service

```typescript
// src/api/users.ts
import { apiClient } from '@/lib/api/client';
import { User, CreateUserData, UpdateUserData } from '@/types/user';

export interface UsersListParams {
  page?: number;
  limit?: number;
  search?: string;
  role?: string;
  status?: string;
  sortBy?: string;
  sortOrder?: 'asc' | 'desc';
}

export interface UsersListResponse {
  users: User[];
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
}

class UsersApi {
  private readonly basePath = '/users';

  async getUsers(params?: UsersListParams): Promise<UsersListResponse> {
    const response = await apiClient.get<UsersListResponse>(this.basePath, {
      params,
    });
    return response.data;
  }

  async getUser(id: string): Promise<User> {
    const response = await apiClient.get<User>(`${this.basePath}/${id}`);
    return response.data;
  }

  async createUser(data: CreateUserData): Promise<User> {
    const response = await apiClient.post<User>(this.basePath, data);
    return response.data;
  }

  async updateUser(id: string, data: UpdateUserData): Promise<User> {
    const response = await apiClient.patch<User>(`${this.basePath}/${id}`, data);
    return response.data;
  }

  async deleteUser(id: string): Promise<void> {
    await apiClient.delete(`${this.basePath}/${id}`);
  }

  async updateUserStatus(id: string, status: 'active' | 'inactive'): Promise<User> {
    const response = await apiClient.patch<User>(`${this.basePath}/${id}/status`, {
      status,
    });
    return response.data;
  }

  async updateUserRole(id: string, role: string): Promise<User> {
    const response = await apiClient.patch<User>(`${this.basePath}/${id}/role`, {
      role,
    });
    return response.data;
  }

  async bulkUpdateUsers(
    userIds: string[],
    updates: Partial<UpdateUserData>
  ): Promise<User[]> {
    const response = await apiClient.patch<User[]>(`${this.basePath}/bulk`, {
      userIds,
      updates,
    });
    return response.data;
  }

  async bulkDeleteUsers(userIds: string[]): Promise<void> {
    await apiClient.delete(`${this.basePath}/bulk`, {
      data: { userIds },
    });
  }

  async exportUsers(params?: UsersListParams): Promise<Blob> {
    const response = await apiClient.get(`${this.basePath}/export`, {
      params,
      responseType: 'blob',
    });
    return response.data;
  }

  async importUsers(file: File): Promise<{ imported: number; errors: any[] }> {
    const response = await apiClient.upload<{ imported: number; errors: any[] }>(
      `${this.basePath}/import`,
      file
    );
    return response.data;
  }
}

export const usersApi = new UsersApi();
```

## React Query Integration

### Query Configuration and Hooks

```typescript
// src/hooks/api/useAuth.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { useNavigate } from 'react-router-dom';
import { authApi, LoginCredentials, RegisterData } from '@/api/auth';
import { useAuthStore } from '@/stores/authStore';
import { useAppStore } from '@/stores/appStore';

export const useLogin = () => {
  const navigate = useNavigate();
  const { login: setAuthState } = useAuthStore();
  const { addNotification } = useAppStore();

  return useMutation({
    mutationFn: (credentials: LoginCredentials) => authApi.login(credentials),
    onSuccess: (data) => {
      // Update auth store
      setAuthState(data);
      
      // Set token in API client
      authApi.setToken(data.token);
      
      // Show success notification
      addNotification({
        type: 'success',
        title: 'Welcome back!',
        message: `Hello ${data.user.firstName}, you've successfully logged in.`,
      });

      // Navigate to dashboard
      navigate('/dashboard');
    },
    onError: (error: any) => {
      addNotification({
        type: 'error',
        title: 'Login Failed',
        message: error.message || 'Invalid email or password.',
      });
    },
  });
};

export const useRegister = () => {
  const navigate = useNavigate();
  const { addNotification } = useAppStore();

  return useMutation({
    mutationFn: (data: RegisterData) => authApi.register(data),
    onSuccess: (data) => {
      addNotification({
        type: 'success',
        title: 'Account Created',
        message: 'Your account has been created successfully. Please verify your email.',
      });

      // Navigate to verification page
      navigate('/auth/verify-email', { 
        state: { email: data.user.email } 
      });
    },
    onError: (error: any) => {
      addNotification({
        type: 'error',
        title: 'Registration Failed',
        message: error.message || 'Failed to create account.',
      });
    },
  });
};

export const useLogout = () => {
  const queryClient = useQueryClient();
  const navigate = useNavigate();
  const { logout: clearAuthState } = useAuthStore();
  const { addNotification } = useAppStore();

  return useMutation({
    mutationFn: () => authApi.logout(),
    onSuccess: () => {
      // Clear auth state
      clearAuthState();
      
      // Clear token from API client
      authApi.clearToken();
      
      // Clear all cached data
      queryClient.clear();
      
      // Show notification
      addNotification({
        type: 'info',
        title: 'Logged Out',
        message: 'You have been successfully logged out.',
      });

      // Navigate to home page
      navigate('/');
    },
    onError: (error: any) => {
      // Even if logout fails on server, clear local state
      clearAuthState();
      authApi.clearToken();
      queryClient.clear();
      navigate('/');
    },
  });
};

export const useForgotPassword = () => {
  const { addNotification } = useAppStore();

  return useMutation({
    mutationFn: (email: string) => authApi.forgotPassword(email),
    onSuccess: () => {
      addNotification({
        type: 'success',
        title: 'Reset Email Sent',
        message: 'Please check your email for password reset instructions.',
      });
    },
    onError: (error: any) => {
      addNotification({
        type: 'error',
        title: 'Reset Failed',
        message: error.message || 'Failed to send reset email.',
      });
    },
  });
};
```

### Optimistic Updates Implementation

```typescript
// src/hooks/api/useOptimisticUpdates.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { usersApi } from '@/api/users';
import { User } from '@/types/user';
import { userKeys } from './useUsers';

export const useOptimisticUserUpdate = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ id, data }: { id: string; data: Partial<User> }) =>
      usersApi.updateUser(id, data),
    
    onMutate: async ({ id, data }) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: userKeys.detail(id) });
      await queryClient.cancelQueries({ queryKey: userKeys.lists() });

      // Snapshot previous values
      const previousUser = queryClient.getQueryData<User>(userKeys.detail(id));
      const previousUsersList = queryClient.getQueriesData({ queryKey: userKeys.lists() });

      // Optimistically update user detail
      if (previousUser) {
        queryClient.setQueryData(userKeys.detail(id), {
          ...previousUser,
          ...data,
          updatedAt: new Date().toISOString(),
        });
      }

      // Optimistically update users in lists
      previousUsersList.forEach(([queryKey, queryData]) => {
        if (queryData?.users) {
          const updatedUsers = queryData.users.map((user: User) =>
            user.id === id ? { ...user, ...data, updatedAt: new Date().toISOString() } : user
          );
          queryClient.setQueryData(queryKey, {
            ...queryData,
            users: updatedUsers,
          });
        }
      });

      return { previousUser, previousUsersList };
    },

    onError: (error, { id }, context) => {
      // Rollback optimistic updates
      if (context?.previousUser) {
        queryClient.setQueryData(userKeys.detail(id), context.previousUser);
      }

      if (context?.previousUsersList) {
        context.previousUsersList.forEach(([queryKey, queryData]) => {
          queryClient.setQueryData(queryKey, queryData);
        });
      }
    },

    onSettled: (_, __, { id }) => {
      // Refetch to ensure consistency
      queryClient.invalidateQueries({ queryKey: userKeys.detail(id) });
      queryClient.invalidateQueries({ queryKey: userKeys.lists() });
    },
  });
};
```

## Real-time Communication

### WebSocket Service

```typescript
// src/services/websocket.ts
import { useAuthStore } from '@/stores/authStore';
import { useAppStore } from '@/stores/appStore';

export interface WebSocketMessage {
  type: string;
  payload: any;
  timestamp: string;
  id: string;
}

export interface WebSocketConfig {
  url: string;
  reconnectInterval: number;
  maxReconnectAttempts: number;
  heartbeatInterval: number;
}

class WebSocketService {
  private ws: WebSocket | null = null;
  private config: WebSocketConfig;
  private reconnectAttempts = 0;
  private reconnectTimer: NodeJS.Timeout | null = null;
  private heartbeatTimer: NodeJS.Timeout | null = null;
  private eventListeners: Map<string, Set<Function>> = new Map();
  private isConnecting = false;
  private isAuthenticated = false;

  constructor(config: WebSocketConfig) {
    this.config = config;
  }

  connect(userId?: string): Promise<void> {
    return new Promise((resolve, reject) => {
      if (this.ws?.readyState === WebSocket.OPEN) {
        resolve();
        return;
      }

      if (this.isConnecting) {
        reject(new Error('Connection already in progress'));
        return;
      }

      this.isConnecting = true;
      
      try {
        const token = useAuthStore.getState().token;
        const wsUrl = `${this.config.url}?token=${token}&userId=${userId}`;
        
        this.ws = new WebSocket(wsUrl);

        this.ws.onopen = () => {
          console.log('WebSocket connected');
          this.isConnecting = false;
          this.isAuthenticated = true;
          this.reconnectAttempts = 0;
          this.startHeartbeat();
          
          useAppStore.getState().addNotification({
            type: 'success',
            title: 'Connected',
            message: 'Real-time connection established.',
          });

          resolve();
        };

        this.ws.onmessage = (event) => {
          this.handleMessage(event);
        };

        this.ws.onclose = (event) => {
          console.log('WebSocket disconnected:', event.code, event.reason);
          this.isConnecting = false;
          this.isAuthenticated = false;
          this.stopHeartbeat();
          
          if (!event.wasClean && this.shouldReconnect()) {
            this.scheduleReconnect();
          }
        };

        this.ws.onerror = (error) => {
          console.error('WebSocket error:', error);
          this.isConnecting = false;
          
          useAppStore.getState().addNotification({
            type: 'error',
            title: 'Connection Error',
            message: 'Failed to establish real-time connection.',
          });

          reject(error);
        };
      } catch (error) {
        this.isConnecting = false;
        reject(error);
      }
    });
  }

  disconnect(): void {
    if (this.reconnectTimer) {
      clearTimeout(this.reconnectTimer);
      this.reconnectTimer = null;
    }

    this.stopHeartbeat();

    if (this.ws) {
      this.ws.close(1000, 'Client disconnect');
      this.ws = null;
    }

    this.isAuthenticated = false;
    this.reconnectAttempts = 0;
  }

  send(type: string, payload: any): void {
    if (!this.isConnected()) {
      throw new Error('WebSocket not connected');
    }

    const message: WebSocketMessage = {
      type,
      payload,
      timestamp: new Date().toISOString(),
      id: this.generateMessageId(),
    };

    this.ws!.send(JSON.stringify(message));
  }

  on(event: string, callback: Function): void {
    if (!this.eventListeners.has(event)) {
      this.eventListeners.set(event, new Set());
    }
    this.eventListeners.get(event)!.add(callback);
  }

  off(event: string, callback: Function): void {
    const listeners = this.eventListeners.get(event);
    if (listeners) {
      listeners.delete(callback);
      if (listeners.size === 0) {
        this.eventListeners.delete(event);
      }
    }
  }

  private handleMessage(event: MessageEvent): void {
    try {
      const message: WebSocketMessage = JSON.parse(event.data);
      
      // Handle system messages
      if (message.type === 'heartbeat') {
        this.send('heartbeat-ack', {});
        return;
      }

      // Emit to listeners
      const listeners = this.eventListeners.get(message.type);
      if (listeners) {
        listeners.forEach(callback => {
          try {
            callback(message.payload);
          } catch (error) {
            console.error('Error in WebSocket event listener:', error);
          }
        });
      }

      // Emit to global listeners
      const globalListeners = this.eventListeners.get('*');
      if (globalListeners) {
        globalListeners.forEach(callback => {
          try {
            callback(message);
          } catch (error) {
            console.error('Error in global WebSocket listener:', error);
          }
        });
      }
    } catch (error) {
      console.error('Error parsing WebSocket message:', error);
    }
  }

  private startHeartbeat(): void {
    this.heartbeatTimer = setInterval(() => {
      if (this.isConnected()) {
        this.send('heartbeat', { timestamp: Date.now() });
      }
    }, this.config.heartbeatInterval);
  }

  private stopHeartbeat(): void {
    if (this.heartbeatTimer) {
      clearInterval(this.heartbeatTimer);
      this.heartbeatTimer = null;
    }
  }

  private shouldReconnect(): boolean {
    return (
      this.reconnectAttempts < this.config.maxReconnectAttempts &&
      useAuthStore.getState().isAuthenticated
    );
  }

  private scheduleReconnect(): void {
    if (this.reconnectTimer) {
      clearTimeout(this.reconnectTimer);
    }

    const delay = Math.min(
      1000 * Math.pow(2, this.reconnectAttempts),
      30000
    );

    this.reconnectTimer = setTimeout(() => {
      this.reconnectAttempts++;
      console.log(`Attempting to reconnect (${this.reconnectAttempts}/${this.config.maxReconnectAttempts})`);
      
      const userId = useAuthStore.getState().user?.id;
      this.connect(userId).catch(() => {
        if (this.shouldReconnect()) {
          this.scheduleReconnect();
        }
      });
    }, delay);
  }

  private isConnected(): boolean {
    return this.ws?.readyState === WebSocket.OPEN && this.isAuthenticated;
  }

  private generateMessageId(): string {
    return `msg_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }

  getConnectionState(): string {
    if (!this.ws) return 'disconnected';
    
    switch (this.ws.readyState) {
      case WebSocket.CONNECTING: return 'connecting';
      case WebSocket.OPEN: return this.isAuthenticated ? 'connected' : 'authenticating';
      case WebSocket.CLOSING: return 'closing';
      case WebSocket.CLOSED: return 'disconnected';
      default: return 'unknown';
    }
  }
}

// Create and export WebSocket service instance
export const websocketService = new WebSocketService({
  url: process.env.REACT_APP_WS_URL || 'ws://localhost:3001/ws',
  reconnectInterval: 5000,
  maxReconnectAttempts: 5,
  heartbeatInterval: 30000,
});
```

## File Upload Implementation

### File Upload Service

```typescript
// src/services/fileUpload.ts
import { apiClient } from '@/lib/api/client';

export interface UploadProgress {
  loaded: number;
  total: number;
  percentage: number;
}

export interface UploadResult {
  id: string;
  filename: string;
  originalName: string;
  mimeType: string;
  size: number;
  url: string;
  thumbnailUrl?: string;
}

export interface UploadOptions {
  onProgress?: (progress: UploadProgress) => void;
  onSuccess?: (result: UploadResult) => void;
  onError?: (error: Error) => void;
  maxSize?: number;
  allowedTypes?: string[];
  generateThumbnail?: boolean;
}

class FileUploadService {
  private readonly maxFileSize = 10 * 1024 * 1024; // 10MB
  private readonly allowedTypes = [
    'image/jpeg',
    'image/png',
    'image/gif',
    'image/webp',
    'application/pdf',
    'text/plain',
    'application/msword',
    'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
  ];

  async uploadFile(
    file: File,
    endpoint: string = '/files/upload',
    options: UploadOptions = {}
  ): Promise<UploadResult> {
    // Validate file
    this.validateFile(file, options);

    const formData = new FormData();
    formData.append('file', file);
    
    if (options.generateThumbnail) {
      formData.append('generateThumbnail', 'true');
    }

    try {
      const response = await apiClient.post<UploadResult>(endpoint, formData, {
        headers: {
          'Content-Type': 'multipart/form-data',
        },
        onUploadProgress: (progressEvent) => {
          if (options.onProgress && progressEvent.total) {
            const progress: UploadProgress = {
              loaded: progressEvent.loaded,
              total: progressEvent.total,
              percentage: Math.round((progressEvent.loaded * 100) / progressEvent.total),
            };
            options.onProgress(progress);
          }
        },
      });

      const result = response.data;
      options.onSuccess?.(result);
      return result;
    } catch (error) {
      const uploadError = new Error(
        error instanceof Error ? error.message : 'Upload failed'
      );
      options.onError?.(uploadError);
      throw uploadError;
    }
  }

  async uploadMultipleFiles(
    files: File[],
    endpoint: string = '/files/upload-multiple',
    options: UploadOptions = {}
  ): Promise<UploadResult[]> {
    const formData = new FormData();
    
    files.forEach((file, index) => {
      this.validateFile(file, options);
      formData.append(`files`, file);
    });

    if (options.generateThumbnail) {
      formData.append('generateThumbnail', 'true');
    }

    try {
      const response = await apiClient.post<UploadResult[]>(endpoint, formData, {
        headers: {
          'Content-Type': 'multipart/form-data',
        },
        onUploadProgress: (progressEvent) => {
          if (options.onProgress && progressEvent.total) {
            const progress: UploadProgress = {
              loaded: progressEvent.loaded,
              total: progressEvent.total,
              percentage: Math.round((progressEvent.loaded * 100) / progressEvent.total),
            };
            options.onProgress(progress);
          }
        },
      });

      const results = response.data;
      options.onSuccess?.(results as any);
      return results;
    } catch (error) {
      const uploadError = new Error(
        error instanceof Error ? error.message : 'Upload failed'
      );
      options.onError?.(uploadError);
      throw uploadError;
    }
  }

  async deleteFile(fileId: string): Promise<void> {
    await apiClient.delete(`/files/${fileId}`);
  }

  private validateFile(file: File, options: UploadOptions): void {
    const maxSize = options.maxSize || this.maxFileSize;
    const allowedTypes = options.allowedTypes || this.allowedTypes;

    if (file.size > maxSize) {
      throw new Error(`File size exceeds ${this.formatFileSize(maxSize)} limit`);
    }

    if (!allowedTypes.includes(file.type)) {
      throw new Error(`File type ${file.type} is not allowed`);
    }
  }

  private formatFileSize(bytes: number): string {
    if (bytes === 0) return '0 Bytes';
    
    const k = 1024;
    const sizes = ['Bytes', 'KB', 'MB', 'GB'];
    const i = Math.floor(Math.log(bytes) / Math.log(k));
    
    return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
  }

  // Utility methods
  isImageFile(file: File): boolean {
    return file.type.startsWith('image/');
  }

  isPdfFile(file: File): boolean {
    return file.type === 'application/pdf';
  }

  isDocumentFile(file: File): boolean {
    const documentTypes = [
      'application/pdf',
      'text/plain',
      'application/msword',
      'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
    ];
    return documentTypes.includes(file.type);
  }
}

export const fileUploadService = new FileUploadService();
```

## Error Handling and Retry Logic

### Advanced Error Handling

```typescript
// src/utils/errorHandling.ts
import { AxiosError } from 'axios';
import { ApiError } from '@/lib/api/client';

export interface RetryConfig {
  maxAttempts: number;
  baseDelay: number;
  maxDelay: number;
  backoffFactor: number;
  retryCondition?: (error: any) => boolean;
}

export class ApiErrorHandler {
  private static defaultRetryConfig: RetryConfig = {
    maxAttempts: 3,
    baseDelay: 1000,
    maxDelay: 10000,
    backoffFactor: 2,
    retryCondition: (error: ApiError) => {
      // Retry on network errors and 5xx server errors
      return error.status === 0 || (error.status >= 500 && error.status < 600);
    },
  };

  static async withRetry<T>(
    operation: () => Promise<T>,
    config: Partial<RetryConfig> = {}
  ): Promise<T> {
    const retryConfig = { ...this.defaultRetryConfig, ...config };
    let lastError: any;

    for (let attempt = 1; attempt <= retryConfig.maxAttempts; attempt++) {
      try {
        return await operation();
      } catch (error) {
        lastError = error;

        // Don't retry if this is the last attempt
        if (attempt === retryConfig.maxAttempts) {
          break;
        }

        // Check if we should retry this error
        if (retryConfig.retryCondition && !retryConfig.retryCondition(error)) {
          break;
        }

        // Calculate delay with exponential backoff
        const delay = Math.min(
          retryConfig.baseDelay * Math.pow(retryConfig.backoffFactor, attempt - 1),
          retryConfig.maxDelay
        );

        console.log(`Retrying operation (attempt ${attempt + 1}/${retryConfig.maxAttempts}) after ${delay}ms`);
        
        await this.delay(delay);
      }
    }

    throw lastError;
  }

  static async withCircuitBreaker<T>(
    operation: () => Promise<T>,
    circuitBreakerConfig: {
      failureThreshold: number;
      resetTimeout: number;
      monitoringPeriod: number;
    }
  ): Promise<T> {
    // Circuit breaker implementation would go here
    // For now, just execute the operation
    return operation();
  }

  private static delay(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }

  static categorizeError(error: any): 'network' | 'client' | 'server' | 'unknown' {
    if (!error.status) return 'network';
    if (error.status >= 400 && error.status < 500) return 'client';
    if (error.status >= 500) return 'server';
    return 'unknown';
  }

  static isRetryableError(error: any): boolean {
    const category = this.categorizeError(error);
    return category === 'network' || category === 'server';
  }
}
```

## Testing API Integration

### API Testing Utilities

```typescript
// src/utils/testing/apiTestUtils.ts
import { rest } from 'msw';
import { setupServer } from 'msw/node';
import { ApiResponse } from '@/lib/api/client';

// Mock API responses
export const createMockApiResponse = <T>(data: T, meta?: any): ApiResponse<T> => ({
  data,
  success: true,
  message: 'Success',
  meta: {
    timestamp: new Date().toISOString(),
    requestId: 'test-request-id',
    ...meta,
  },
});

// Mock server setup
export const createMockServer = (handlers: any[] = []) => {
  return setupServer(...handlers);
};

// Common mock handlers
export const mockHandlers = {
  auth: {
    login: rest.post('/api/auth/login', (req, res, ctx) => {
      return res(
        ctx.json(createMockApiResponse({
          user: {
            id: '1',
            email: 'test@example.com',
            firstName: 'Test',
            lastName: 'User',
          },
          token: 'mock-token',
          refreshToken: 'mock-refresh-token',
          expiresIn: 3600,
        }))
      );
    }),

    logout: rest.post('/api/auth/logout', (req, res, ctx) => {
      return res(ctx.json(createMockApiResponse({})));
    }),
  },

  users: {
    getUsers: rest.get('/api/users', (req, res, ctx) => {
      return res(
        ctx.json(createMockApiResponse({
          users: [
            { id: '1', email: 'user1@example.com', firstName: 'User', lastName: 'One' },
            { id: '2', email: 'user2@example.com', firstName: 'User', lastName: 'Two' },
          ],
          pagination: {
            page: 1,
            limit: 10,
            total: 2,
            totalPages: 1,
          },
        }))
      );
    }),

    getUser: rest.get('/api/users/:id', (req, res, ctx) => {
      const { id } = req.params;
      return res(
        ctx.json(createMockApiResponse({
          id,
          email: `user${id}@example.com`,
          firstName: 'Test',
          lastName: 'User',
        }))
      );
    }),
  },
};

// Test utilities
export const waitForApiCall = (timeout = 5000) => {
  return new Promise((resolve, reject) => {
    const timer = setTimeout(() => {
      reject(new Error('API call timeout'));
    }, timeout);

    // In a real implementation, you'd listen for the actual API call
    setTimeout(() => {
      clearTimeout(timer);
      resolve(true);
    }, 100);
  });
};
```

## Best Practices

### 1. API Client Configuration
- **Centralized Configuration**: Single API client with consistent configuration
- **Interceptors**: Request/response interceptors for authentication and error handling
- **Type Safety**: Comprehensive TypeScript interfaces for all API responses
- **Error Standardization**: Consistent error handling and user feedback

### 2. State Management Integration
- **React Query**: Server state management with caching and synchronization
- **Optimistic Updates**: Immediate UI updates with rollback on errors
- **Background Sync**: Automatic data synchronization and cache invalidation
- **Offline Support**: Graceful handling of network connectivity issues

### 3. Real-time Communication
- **WebSocket Management**: Robust connection handling with reconnection logic
- **Event-driven Architecture**: Clean separation of concerns with event listeners
- **Heartbeat Monitoring**: Connection health monitoring and automatic recovery
- **Authentication Integration**: Secure WebSocket connections with token validation

### 4. Performance Optimization
- **Request Deduplication**: Prevent duplicate API calls
- **Response Caching**: Intelligent caching strategies with TTL
- **Bundle Optimization**: Code splitting for API services
- **Error Recovery**: Retry logic and circuit breaker patterns

This comprehensive API integration guide provides the foundation for building robust, scalable, and maintainable API integration in DafnckMachine v3.1. 