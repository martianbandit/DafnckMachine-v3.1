# Error Handling Implementation - DafnckMachine v3.1

## Document Metadata
- **Document Type**: Implementation Guide
- **Version**: 3.1.0
- **Last Updated**: 2025-01-27
- **Related Workflow**: [Frontend_Development.md](mdc:01_Machine/01_Workflow/Phase%204:%20Development%20&%20Quality%20Assurance/12_Frontend_Development.md)
- **Prerequisites**: [Frontend_Security_Implementation.md](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Frontend_Security_Implementation.md)

## Overview

This guide provides comprehensive error handling implementation strategies for DafnckMachine v3.1, covering error boundaries, global error handling, API error management, user-friendly error messages, error recovery strategies, and monitoring integration to ensure robust application reliability.

## Table of Contents

1. [Error Handling Strategy](#error-handling-strategy)
2. [Error Boundaries](#error-boundaries)
3. [Global Error Handling](#global-error-handling)
4. [API Error Management](#api-error-management)
5. [User-Friendly Error Messages](#user-friendly-error-messages)
6. [Error Recovery Strategies](#error-recovery-strategies)

## Error Handling Strategy

### Error Classification

```typescript
// src/types/errors.ts
export enum ErrorType {
  NETWORK = 'NETWORK',
  VALIDATION = 'VALIDATION',
  AUTHENTICATION = 'AUTHENTICATION',
  AUTHORIZATION = 'AUTHORIZATION',
  NOT_FOUND = 'NOT_FOUND',
  SERVER = 'SERVER',
  CLIENT = 'CLIENT',
  UNKNOWN = 'UNKNOWN'
}

export enum ErrorSeverity {
  LOW = 'LOW',
  MEDIUM = 'MEDIUM',
  HIGH = 'HIGH',
  CRITICAL = 'CRITICAL'
}

export interface AppError {
  id: string;
  type: ErrorType;
  severity: ErrorSeverity;
  message: string;
  userMessage: string;
  code?: string;
  details?: Record<string, any>;
  timestamp: number;
  stack?: string;
  context?: {
    userId?: string;
    sessionId?: string;
    url?: string;
    userAgent?: string;
  };
}

export interface ErrorAction {
  type: 'RETRY' | 'RELOAD' | 'NAVIGATE' | 'DISMISS' | 'CONTACT_SUPPORT';
  label: string;
  handler: () => void;
}
```

### Error Factory

```typescript
// src/utils/errorFactory.ts
import { v4 as uuidv4 } from 'uuid';
import { AppError, ErrorType, ErrorSeverity } from '../types/errors';

class ErrorFactory {
  static createError(
    type: ErrorType,
    message: string,
    userMessage: string,
    severity: ErrorSeverity = ErrorSeverity.MEDIUM,
    details?: Record<string, any>
  ): AppError {
    return {
      id: uuidv4(),
      type,
      severity,
      message,
      userMessage,
      details,
      timestamp: Date.now(),
      stack: new Error().stack,
      context: {
        userId: this.getCurrentUserId(),
        sessionId: this.getSessionId(),
        url: window.location.href,
        userAgent: navigator.userAgent
      }
    };
  }

  static createNetworkError(message: string, status?: number): AppError {
    const userMessage = status === 0 
      ? 'Unable to connect to the server. Please check your internet connection.'
      : `Network error occurred. Please try again. (${status})`;

    return this.createError(
      ErrorType.NETWORK,
      message,
      userMessage,
      ErrorSeverity.HIGH,
      { status }
    );
  }

  static createValidationError(field: string, message: string): AppError {
    return this.createError(
      ErrorType.VALIDATION,
      `Validation failed for ${field}: ${message}`,
      `Please check your ${field} and try again.`,
      ErrorSeverity.LOW,
      { field }
    );
  }

  static createAuthenticationError(): AppError {
    return this.createError(
      ErrorType.AUTHENTICATION,
      'Authentication failed',
      'Your session has expired. Please log in again.',
      ErrorSeverity.HIGH
    );
  }

  static createAuthorizationError(): AppError {
    return this.createError(
      ErrorType.AUTHORIZATION,
      'Authorization failed',
      'You do not have permission to perform this action.',
      ErrorSeverity.MEDIUM
    );
  }

  static createNotFoundError(resource: string): AppError {
    return this.createError(
      ErrorType.NOT_FOUND,
      `Resource not found: ${resource}`,
      'The requested item could not be found.',
      ErrorSeverity.MEDIUM,
      { resource }
    );
  }

  static createServerError(message: string): AppError {
    return this.createError(
      ErrorType.SERVER,
      message,
      'A server error occurred. Our team has been notified.',
      ErrorSeverity.HIGH
    );
  }

  private static getCurrentUserId(): string | undefined {
    // Get from auth context or local storage
    return localStorage.getItem('userId') || undefined;
  }

  private static getSessionId(): string | undefined {
    return sessionStorage.getItem('sessionId') || undefined;
  }
}

export { ErrorFactory };
```

## Error Boundaries

### React Error Boundary

```typescript
// src/components/ErrorBoundary.tsx
import React, { Component, ErrorInfo, ReactNode } from 'react';
import { AppError, ErrorType, ErrorSeverity } from '../types/errors';
import { ErrorFactory } from '../utils/errorFactory';
import { errorReportingService } from '../services/errorReporting';
import { ErrorDisplay } from './ErrorDisplay';

interface Props {
  children: ReactNode;
  fallback?: (error: AppError, retry: () => void) => ReactNode;
  onError?: (error: AppError, errorInfo: ErrorInfo) => void;
  level?: 'page' | 'component' | 'critical';
}

interface State {
  hasError: boolean;
  error?: AppError;
  errorInfo?: ErrorInfo;
}

export class ErrorBoundary extends Component<Props, State> {
  private retryTimeoutId?: NodeJS.Timeout;

  constructor(props: Props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error): State {
    const appError = ErrorFactory.createError(
      ErrorType.CLIENT,
      error.message,
      'Something went wrong. Please try refreshing the page.',
      ErrorSeverity.HIGH,
      { originalError: error.name }
    );

    return {
      hasError: true,
      error: appError
    };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    const appError = this.state.error!;
    
    // Add React error info to context
    appError.details = {
      ...appError.details,
      componentStack: errorInfo.componentStack,
      errorBoundary: this.props.level || 'component'
    };

    // Report error
    errorReportingService.reportError(appError);

    // Call custom error handler
    this.props.onError?.(appError, errorInfo);

    // Set error info in state
    this.setState({ errorInfo });
  }

  private handleRetry = () => {
    this.setState({ hasError: false, error: undefined, errorInfo: undefined });
    
    // Clear any existing retry timeout
    if (this.retryTimeoutId) {
      clearTimeout(this.retryTimeoutId);
    }
  };

  private handleReload = () => {
    window.location.reload();
  };

  private renderFallback() {
    const { error } = this.state;
    if (!error) return null;

    if (this.props.fallback) {
      return this.props.fallback(error, this.handleRetry);
    }

    const actions = [
      {
        type: 'RETRY' as const,
        label: 'Try Again',
        handler: this.handleRetry
      },
      {
        type: 'RELOAD' as const,
        label: 'Reload Page',
        handler: this.handleReload
      }
    ];

    return (
      <ErrorDisplay
        error={error}
        actions={actions}
        level={this.props.level}
      />
    );
  }

  render() {
    if (this.state.hasError) {
      return this.renderFallback();
    }

    return this.props.children;
  }
}
```

### Specialized Error Boundaries

```typescript
// src/components/AsyncErrorBoundary.tsx
import React, { ReactNode } from 'react';
import { ErrorBoundary } from './ErrorBoundary';
import { AppError } from '../types/errors';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
}

export const AsyncErrorBoundary: React.FC<Props> = ({ children, fallback }) => {
  const handleAsyncError = (error: AppError) => {
    // Handle async errors specifically
    console.error('Async error caught:', error);
  };

  return (
    <ErrorBoundary
      level="component"
      onError={handleAsyncError}
      fallback={fallback ? () => fallback : undefined}
    >
      {children}
    </ErrorBoundary>
  );
};

// src/components/RouteErrorBoundary.tsx
import React, { ReactNode } from 'react';
import { useNavigate } from 'react-router-dom';
import { ErrorBoundary } from './ErrorBoundary';
import { AppError, ErrorAction } from '../types/errors';

interface Props {
  children: ReactNode;
}

export const RouteErrorBoundary: React.FC<Props> = ({ children }) => {
  const navigate = useNavigate();

  const handleRouteError = (error: AppError) => {
    // Log route-specific error context
    console.error('Route error:', {
      error,
      route: window.location.pathname
    });
  };

  const renderRouteFallback = (error: AppError, retry: () => void) => {
    const actions: ErrorAction[] = [
      {
        type: 'RETRY',
        label: 'Try Again',
        handler: retry
      },
      {
        type: 'NAVIGATE',
        label: 'Go Home',
        handler: () => navigate('/')
      }
    ];

    return (
      <div className="min-h-screen flex items-center justify-center bg-gray-50">
        <div className="max-w-md w-full bg-white shadow-lg rounded-lg p-6">
          <div className="flex items-center mb-4">
            <div className="flex-shrink-0">
              <svg className="h-8 w-8 text-red-400" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-2.5L13.732 4c-.77-.833-1.964-.833-2.732 0L3.732 16.5c-.77.833.192 2.5 1.732 2.5z" />
              </svg>
            </div>
            <div className="ml-3">
              <h3 className="text-lg font-medium text-gray-900">
                Page Error
              </h3>
            </div>
          </div>
          
          <div className="mb-4">
            <p className="text-sm text-gray-600">
              {error.userMessage}
            </p>
          </div>
          
          <div className="flex space-x-3">
            {actions.map((action, index) => (
              <button
                key={index}
                onClick={action.handler}
                className="flex-1 bg-blue-600 text-white px-4 py-2 rounded-md text-sm font-medium hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500"
              >
                {action.label}
              </button>
            ))}
          </div>
        </div>
      </div>
    );
  };

  return (
    <ErrorBoundary
      level="page"
      onError={handleRouteError}
      fallback={renderRouteFallback}
    >
      {children}
    </ErrorBoundary>
  );
};
```

## Global Error Handling

### Global Error Handler

```typescript
// src/utils/globalErrorHandler.ts
import { AppError, ErrorType, ErrorSeverity } from '../types/errors';
import { ErrorFactory } from './errorFactory';
import { errorReportingService } from '../services/errorReporting';
import { notificationService } from '../services/notification';

class GlobalErrorHandler {
  private errorQueue: AppError[] = [];
  private isProcessing = false;

  constructor() {
    this.setupGlobalHandlers();
  }

  private setupGlobalHandlers() {
    // Handle unhandled promise rejections
    window.addEventListener('unhandledrejection', (event) => {
      const error = ErrorFactory.createError(
        ErrorType.CLIENT,
        `Unhandled promise rejection: ${event.reason}`,
        'An unexpected error occurred. Please try again.',
        ErrorSeverity.HIGH,
        { reason: event.reason }
      );

      this.handleError(error);
      event.preventDefault();
    });

    // Handle global JavaScript errors
    window.addEventListener('error', (event) => {
      const error = ErrorFactory.createError(
        ErrorType.CLIENT,
        event.message,
        'An unexpected error occurred. Please refresh the page.',
        ErrorSeverity.HIGH,
        {
          filename: event.filename,
          lineno: event.lineno,
          colno: event.colno
        }
      );

      this.handleError(error);
    });

    // Handle resource loading errors
    window.addEventListener('error', (event) => {
      if (event.target !== window) {
        const target = event.target as HTMLElement;
        const error = ErrorFactory.createError(
          ErrorType.CLIENT,
          `Resource failed to load: ${target.tagName}`,
          'Some content failed to load. Please refresh the page.',
          ErrorSeverity.MEDIUM,
          {
            tagName: target.tagName,
            src: (target as any).src || (target as any).href
          }
        );

        this.handleError(error);
      }
    }, true);
  }

  handleError(error: AppError): void {
    // Add to queue
    this.errorQueue.push(error);

    // Process queue if not already processing
    if (!this.isProcessing) {
      this.processErrorQueue();
    }
  }

  private async processErrorQueue(): Promise<void> {
    this.isProcessing = true;

    while (this.errorQueue.length > 0) {
      const error = this.errorQueue.shift()!;
      await this.processError(error);
    }

    this.isProcessing = false;
  }

  private async processError(error: AppError): Promise<void> {
    try {
      // Report error to monitoring service
      await errorReportingService.reportError(error);

      // Show user notification based on severity
      this.showUserNotification(error);

      // Log to console in development
      if (process.env.NODE_ENV === 'development') {
        console.error('Global error:', error);
      }
    } catch (reportingError) {
      console.error('Failed to process error:', reportingError);
    }
  }

  private showUserNotification(error: AppError): void {
    switch (error.severity) {
      case ErrorSeverity.CRITICAL:
        notificationService.showError(error.userMessage, {
          persistent: true,
          actions: [
            {
              label: 'Reload Page',
              handler: () => window.location.reload()
            }
          ]
        });
        break;

      case ErrorSeverity.HIGH:
        notificationService.showError(error.userMessage, {
          duration: 8000
        });
        break;

      case ErrorSeverity.MEDIUM:
        notificationService.showWarning(error.userMessage, {
          duration: 5000
        });
        break;

      case ErrorSeverity.LOW:
        // Don't show notification for low severity errors
        break;
    }
  }
}

export const globalErrorHandler = new GlobalErrorHandler();
```

### Error Context Provider

```typescript
// src/contexts/ErrorContext.tsx
import React, { createContext, useContext, useReducer, ReactNode } from 'react';
import { AppError } from '../types/errors';
import { globalErrorHandler } from '../utils/globalErrorHandler';

interface ErrorState {
  errors: AppError[];
  isLoading: boolean;
}

type ErrorAction =
  | { type: 'ADD_ERROR'; payload: AppError }
  | { type: 'REMOVE_ERROR'; payload: string }
  | { type: 'CLEAR_ERRORS' }
  | { type: 'SET_LOADING'; payload: boolean };

const initialState: ErrorState = {
  errors: [],
  isLoading: false
};

const errorReducer = (state: ErrorState, action: ErrorAction): ErrorState => {
  switch (action.type) {
    case 'ADD_ERROR':
      return {
        ...state,
        errors: [...state.errors, action.payload]
      };
    case 'REMOVE_ERROR':
      return {
        ...state,
        errors: state.errors.filter(error => error.id !== action.payload)
      };
    case 'CLEAR_ERRORS':
      return {
        ...state,
        errors: []
      };
    case 'SET_LOADING':
      return {
        ...state,
        isLoading: action.payload
      };
    default:
      return state;
  }
};

interface ErrorContextValue {
  state: ErrorState;
  addError: (error: AppError) => void;
  removeError: (errorId: string) => void;
  clearErrors: () => void;
  setLoading: (loading: boolean) => void;
}

const ErrorContext = createContext<ErrorContextValue | undefined>(undefined);

export const useError = () => {
  const context = useContext(ErrorContext);
  if (!context) {
    throw new Error('useError must be used within an ErrorProvider');
  }
  return context;
};

interface Props {
  children: ReactNode;
}

export const ErrorProvider: React.FC<Props> = ({ children }) => {
  const [state, dispatch] = useReducer(errorReducer, initialState);

  const addError = (error: AppError) => {
    dispatch({ type: 'ADD_ERROR', payload: error });
    globalErrorHandler.handleError(error);
  };

  const removeError = (errorId: string) => {
    dispatch({ type: 'REMOVE_ERROR', payload: errorId });
  };

  const clearErrors = () => {
    dispatch({ type: 'CLEAR_ERRORS' });
  };

  const setLoading = (loading: boolean) => {
    dispatch({ type: 'SET_LOADING', payload: loading });
  };

  const value: ErrorContextValue = {
    state,
    addError,
    removeError,
    clearErrors,
    setLoading
  };

  return (
    <ErrorContext.Provider value={value}>
      {children}
    </ErrorContext.Provider>
  );
};
```

## API Error Management

### API Error Handler

```typescript
// src/services/apiErrorHandler.ts
import { AxiosError, AxiosResponse } from 'axios';
import { AppError, ErrorType, ErrorSeverity } from '../types/errors';
import { ErrorFactory } from '../utils/errorFactory';

export interface ApiErrorResponse {
  message: string;
  code?: string;
  details?: Record<string, any>;
  errors?: Array<{
    field: string;
    message: string;
  }>;
}

class ApiErrorHandler {
  handleApiError(error: AxiosError): AppError {
    const response = error.response;
    const request = error.request;

    // Network error
    if (!response && request) {
      return ErrorFactory.createNetworkError(
        'Network request failed',
        0
      );
    }

    // No response received
    if (!response) {
      return ErrorFactory.createError(
        ErrorType.NETWORK,
        'No response received from server',
        'Unable to connect to the server. Please check your connection.',
        ErrorSeverity.HIGH
      );
    }

    // Handle different status codes
    return this.handleStatusCode(response, error);
  }

  private handleStatusCode(response: AxiosResponse, error: AxiosError): AppError {
    const status = response.status;
    const data = response.data as ApiErrorResponse;

    switch (status) {
      case 400:
        return this.handleBadRequest(data);
      case 401:
        return this.handleUnauthorized(data);
      case 403:
        return this.handleForbidden(data);
      case 404:
        return this.handleNotFound(data);
      case 422:
        return this.handleValidationError(data);
      case 429:
        return this.handleRateLimit(data);
      case 500:
      case 502:
      case 503:
      case 504:
        return this.handleServerError(data, status);
      default:
        return this.handleUnknownError(data, status);
    }
  }

  private handleBadRequest(data: ApiErrorResponse): AppError {
    return ErrorFactory.createError(
      ErrorType.VALIDATION,
      data.message || 'Bad request',
      'Please check your input and try again.',
      ErrorSeverity.MEDIUM,
      { apiResponse: data }
    );
  }

  private handleUnauthorized(data: ApiErrorResponse): AppError {
    return ErrorFactory.createAuthenticationError();
  }

  private handleForbidden(data: ApiErrorResponse): AppError {
    return ErrorFactory.createAuthorizationError();
  }

  private handleNotFound(data: ApiErrorResponse): AppError {
    return ErrorFactory.createNotFoundError(
      data.details?.resource || 'resource'
    );
  }

  private handleValidationError(data: ApiErrorResponse): AppError {
    const firstError = data.errors?.[0];
    if (firstError) {
      return ErrorFactory.createValidationError(
        firstError.field,
        firstError.message
      );
    }

    return ErrorFactory.createError(
      ErrorType.VALIDATION,
      data.message || 'Validation failed',
      'Please check your input and try again.',
      ErrorSeverity.MEDIUM,
      { apiResponse: data }
    );
  }

  private handleRateLimit(data: ApiErrorResponse): AppError {
    return ErrorFactory.createError(
      ErrorType.CLIENT,
      'Rate limit exceeded',
      'Too many requests. Please wait a moment and try again.',
      ErrorSeverity.MEDIUM,
      { apiResponse: data }
    );
  }

  private handleServerError(data: ApiErrorResponse, status: number): AppError {
    return ErrorFactory.createServerError(
      data.message || `Server error (${status})`
    );
  }

  private handleUnknownError(data: ApiErrorResponse, status: number): AppError {
    return ErrorFactory.createError(
      ErrorType.UNKNOWN,
      data.message || `Unknown error (${status})`,
      'An unexpected error occurred. Please try again.',
      ErrorSeverity.MEDIUM,
      { status, apiResponse: data }
    );
  }
}

export const apiErrorHandler = new ApiErrorHandler();
```

### API Client with Error Handling

```typescript
// src/services/apiClient.ts
import axios, { AxiosInstance, AxiosRequestConfig, AxiosResponse } from 'axios';
import { apiErrorHandler } from './apiErrorHandler';
import { AppError } from '../types/errors';

class ApiClient {
  private client: AxiosInstance;
  private retryAttempts = 3;
  private retryDelay = 1000;

  constructor() {
    this.client = axios.create({
      baseURL: process.env.REACT_APP_API_URL,
      timeout: 10000,
      headers: {
        'Content-Type': 'application/json'
      }
    });

    this.setupInterceptors();
  }

  private setupInterceptors() {
    // Request interceptor
    this.client.interceptors.request.use(
      (config) => {
        // Add auth token
        const token = localStorage.getItem('authToken');
        if (token) {
          config.headers.Authorization = `Bearer ${token}`;
        }
        return config;
      },
      (error) => Promise.reject(error)
    );

    // Response interceptor
    this.client.interceptors.response.use(
      (response) => response,
      async (error) => {
        const appError = apiErrorHandler.handleApiError(error);
        
        // Handle specific error types
        if (appError.type === 'AUTHENTICATION') {
          this.handleAuthenticationError();
        }

        // Retry logic for network errors
        if (appError.type === 'NETWORK' && this.shouldRetry(error.config)) {
          return this.retryRequest(error.config);
        }

        throw appError;
      }
    );
  }

  private handleAuthenticationError() {
    // Clear auth token
    localStorage.removeItem('authToken');
    
    // Redirect to login
    window.location.href = '/login';
  }

  private shouldRetry(config: AxiosRequestConfig): boolean {
    return (
      config &&
      !config.__retryCount &&
      config.method?.toLowerCase() === 'get'
    );
  }

  private async retryRequest(config: AxiosRequestConfig): Promise<AxiosResponse> {
    config.__retryCount = config.__retryCount || 0;

    if (config.__retryCount >= this.retryAttempts) {
      throw apiErrorHandler.handleApiError({
        message: 'Max retry attempts exceeded',
        config,
        isAxiosError: true,
        toJSON: () => ({})
      } as any);
    }

    config.__retryCount++;

    // Wait before retrying
    await new Promise(resolve => 
      setTimeout(resolve, this.retryDelay * config.__retryCount)
    );

    return this.client.request(config);
  }

  async get<T>(url: string, config?: AxiosRequestConfig): Promise<T> {
    try {
      const response = await this.client.get<T>(url, config);
      return response.data;
    } catch (error) {
      throw error as AppError;
    }
  }

  async post<T>(url: string, data?: any, config?: AxiosRequestConfig): Promise<T> {
    try {
      const response = await this.client.post<T>(url, data, config);
      return response.data;
    } catch (error) {
      throw error as AppError;
    }
  }

  async put<T>(url: string, data?: any, config?: AxiosRequestConfig): Promise<T> {
    try {
      const response = await this.client.put<T>(url, data, config);
      return response.data;
    } catch (error) {
      throw error as AppError;
    }
  }

  async delete<T>(url: string, config?: AxiosRequestConfig): Promise<T> {
    try {
      const response = await this.client.delete<T>(url, config);
      return response.data;
    } catch (error) {
      throw error as AppError;
    }
  }
}

// Extend AxiosRequestConfig to include retry count
declare module 'axios' {
  interface AxiosRequestConfig {
    __retryCount?: number;
  }
}

export const apiClient = new ApiClient();
```

## User-Friendly Error Messages

### Error Display Component

```typescript
// src/components/ErrorDisplay.tsx
import React from 'react';
import { AppError, ErrorAction, ErrorSeverity } from '../types/errors';

interface Props {
  error: AppError;
  actions?: ErrorAction[];
  level?: 'page' | 'component' | 'critical';
  className?: string;
}

export const ErrorDisplay: React.FC<Props> = ({
  error,
  actions = [],
  level = 'component',
  className = ''
}) => {
  const getSeverityStyles = () => {
    switch (error.severity) {
      case ErrorSeverity.CRITICAL:
        return 'border-red-600 bg-red-50 text-red-800';
      case ErrorSeverity.HIGH:
        return 'border-red-400 bg-red-50 text-red-700';
      case ErrorSeverity.MEDIUM:
        return 'border-yellow-400 bg-yellow-50 text-yellow-700';
      case ErrorSeverity.LOW:
        return 'border-blue-400 bg-blue-50 text-blue-700';
      default:
        return 'border-gray-400 bg-gray-50 text-gray-700';
    }
  };

  const getSeverityIcon = () => {
    switch (error.severity) {
      case ErrorSeverity.CRITICAL:
      case ErrorSeverity.HIGH:
        return (
          <svg className="h-5 w-5 text-red-400" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-2.5L13.732 4c-.77-.833-1.964-.833-2.732 0L3.732 16.5c-.77.833.192 2.5 1.732 2.5z" />
          </svg>
        );
      case ErrorSeverity.MEDIUM:
        return (
          <svg className="h-5 w-5 text-yellow-400" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-2.5L13.732 4c-.77-.833-1.964-.833-2.732 0L3.732 16.5c-.77.833.192 2.5 1.732 2.5z" />
          </svg>
        );
      default:
        return (
          <svg className="h-5 w-5 text-blue-400" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
          </svg>
        );
    }
  };

  const getContainerStyles = () => {
    if (level === 'page') {
      return 'min-h-screen flex items-center justify-center p-4';
    }
    return 'w-full';
  };

  return (
    <div className={`${getContainerStyles()} ${className}`}>
      <div className={`border rounded-lg p-4 ${getSeverityStyles()}`}>
        <div className="flex items-start">
          <div className="flex-shrink-0">
            {getSeverityIcon()}
          </div>
          
          <div className="ml-3 flex-1">
            <h3 className="text-sm font-medium">
              {error.severity === ErrorSeverity.CRITICAL ? 'Critical Error' :
               error.severity === ErrorSeverity.HIGH ? 'Error' :
               error.severity === ErrorSeverity.MEDIUM ? 'Warning' : 'Notice'}
            </h3>
            
            <div className="mt-2 text-sm">
              <p>{error.userMessage}</p>
            </div>
            
            {process.env.NODE_ENV === 'development' && (
              <details className="mt-2">
                <summary className="text-xs cursor-pointer">Technical Details</summary>
                <pre className="mt-1 text-xs bg-gray-100 p-2 rounded overflow-auto">
                  {JSON.stringify(error, null, 2)}
                </pre>
              </details>
            )}
            
            {actions.length > 0 && (
              <div className="mt-4 flex space-x-2">
                {actions.map((action, index) => (
                  <button
                    key={index}
                    onClick={action.handler}
                    className="text-sm bg-white border border-gray-300 rounded px-3 py-1 hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-blue-500"
                  >
                    {action.label}
                  </button>
                ))}
              </div>
            )}
          </div>
        </div>
      </div>
    </div>
  );
};
```

### Error Toast Notifications

```typescript
// src/components/ErrorToast.tsx
import React, { useEffect, useState } from 'react';
import { AppError, ErrorSeverity } from '../types/errors';

interface Props {
  error: AppError;
  onDismiss: () => void;
  duration?: number;
}

export const ErrorToast: React.FC<Props> = ({
  error,
  onDismiss,
  duration = 5000
}) => {
  const [isVisible, setIsVisible] = useState(true);

  useEffect(() => {
    if (duration > 0) {
      const timer = setTimeout(() => {
        setIsVisible(false);
        setTimeout(onDismiss, 300); // Allow fade out animation
      }, duration);

      return () => clearTimeout(timer);
    }
  }, [duration, onDismiss]);

  const getSeverityStyles = () => {
    switch (error.severity) {
      case ErrorSeverity.CRITICAL:
      case ErrorSeverity.HIGH:
        return 'bg-red-500 border-red-600';
      case ErrorSeverity.MEDIUM:
        return 'bg-yellow-500 border-yellow-600';
      default:
        return 'bg-blue-500 border-blue-600';
    }
  };

  return (
    <div
      className={`fixed top-4 right-4 max-w-sm w-full transform transition-all duration-300 z-50 ${
        isVisible ? 'translate-x-0 opacity-100' : 'translate-x-full opacity-0'
      }`}
    >
      <div className={`rounded-lg border-l-4 p-4 shadow-lg ${getSeverityStyles()}`}>
        <div className="flex items-start">
          <div className="flex-shrink-0">
            <svg className="h-5 w-5 text-white" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-2.5L13.732 4c-.77-.833-1.964-.833-2.732 0L3.732 16.5c-.77.833.192 2.5 1.732 2.5z" />
            </svg>
          </div>
          
          <div className="ml-3 flex-1">
            <p className="text-sm font-medium text-white">
              {error.userMessage}
            </p>
          </div>
          
          <div className="ml-4 flex-shrink-0">
            <button
              onClick={() => {
                setIsVisible(false);
                setTimeout(onDismiss, 300);
              }}
              className="inline-flex text-white hover:text-gray-200 focus:outline-none focus:text-gray-200"
            >
              <svg className="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M6 18L18 6M6 6l12 12" />
              </svg>
            </button>
          </div>
        </div>
      </div>
    </div>
  );
};
```

## Error Recovery Strategies

### Retry Mechanisms

```typescript
// src/hooks/useRetry.ts
import { useState, useCallback } from 'react';
import { AppError } from '../types/errors';

interface RetryOptions {
  maxAttempts?: number;
  delay?: number;
  backoff?: boolean;
  onRetry?: (attempt: number) => void;
  onMaxAttemptsReached?: () => void;
}

interface RetryState {
  isRetrying: boolean;
  attempts: number;
  lastError?: AppError;
}

export const useRetry = (options: RetryOptions = {}) => {
  const {
    maxAttempts = 3,
    delay = 1000,
    backoff = true,
    onRetry,
    onMaxAttemptsReached
  } = options;

  const [state, setState] = useState<RetryState>({
    isRetrying: false,
    attempts: 0
  });

  const retry = useCallback(async <T>(
    operation: () => Promise<T>
  ): Promise<T> => {
    setState(prev => ({
      ...prev,
      isRetrying: true,
      attempts: 0
    }));

    let lastError: AppError;

    for (let attempt = 1; attempt <= maxAttempts; attempt++) {
      try {
        const result = await operation();
        setState(prev => ({
          ...prev,
          isRetrying: false,
          attempts: attempt
        }));
        return result;
      } catch (error) {
        lastError = error as AppError;
        
        setState(prev => ({
          ...prev,
          attempts: attempt,
          lastError
        }));

        if (attempt < maxAttempts) {
          onRetry?.(attempt);
          
          // Calculate delay with optional backoff
          const currentDelay = backoff ? delay * Math.pow(2, attempt - 1) : delay;
          await new Promise(resolve => setTimeout(resolve, currentDelay));
        }
      }
    }

    setState(prev => ({
      ...prev,
      isRetrying: false
    }));

    onMaxAttemptsReached?.();
    throw lastError!;
  }, [maxAttempts, delay, backoff, onRetry, onMaxAttemptsReached]);

  const reset = useCallback(() => {
    setState({
      isRetrying: false,
      attempts: 0
    });
  }, []);

  return {
    retry,
    reset,
    ...state
  };
};
```

### Error Recovery Hook

```typescript
// src/hooks/useErrorRecovery.ts
import { useCallback } from 'react';
import { useError } from '../contexts/ErrorContext';
import { AppError, ErrorType } from '../types/errors';
import { useRetry } from './useRetry';

export const useErrorRecovery = () => {
  const { addError } = useError();
  const { retry } = useRetry({
    maxAttempts: 3,
    delay: 1000,
    backoff: true
  });

  const withErrorRecovery = useCallback(async <T>(
    operation: () => Promise<T>,
    fallback?: () => T | Promise<T>
  ): Promise<T> => {
    try {
      return await retry(operation);
    } catch (error) {
      const appError = error as AppError;
      
      // Try fallback if available
      if (fallback) {
        try {
          return await fallback();
        } catch (fallbackError) {
          addError(appError);
          throw appError;
        }
      }

      addError(appError);
      throw appError;
    }
  }, [retry, addError]);

  const withNetworkRecovery = useCallback(async <T>(
    operation: () => Promise<T>
  ): Promise<T> => {
    return withErrorRecovery(
      operation,
      async () => {
        // Check if we have cached data
        const cacheKey = operation.toString();
        const cachedData = localStorage.getItem(`cache_${cacheKey}`);
        
        if (cachedData) {
          return JSON.parse(cachedData);
        }
        
        throw new Error('No cached data available');
      }
    );
  }, [withErrorRecovery]);

  return {
    withErrorRecovery,
    withNetworkRecovery
  };
};
```

---

## Summary

This Error Handling Implementation Guide provides comprehensive error management strategies for DafnckMachine v3.1, ensuring robust application reliability through error boundaries, global error handling, API error management, user-friendly error messages, and error recovery strategies.

Key error handling areas include:

1. **Error Boundaries**: React error boundaries for component-level error handling
2. **Global Error Handling**: Centralized error management and reporting
3. **API Error Management**: Structured API error handling with retry logic
4. **User-Friendly Error Messages**: Clear, actionable error displays and notifications
5. **Error Recovery Strategies**: Retry mechanisms and fallback strategies

The guide ensures comprehensive error coverage while maintaining excellent user experience and providing actionable recovery options. 