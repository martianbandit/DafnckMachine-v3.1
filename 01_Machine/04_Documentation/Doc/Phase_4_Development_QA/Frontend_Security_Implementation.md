# Frontend Security Implementation - DafnckMachine v3.1

## Document Metadata
- **Document Type**: Frontend Security Implementation Guide
- **Version**: 3.1.0
- **Last Updated**: 2025-01-27
- **Owner**: Frontend Development Team
- **Status**: Active Development

## Overview

This document provides comprehensive guidance for implementing robust security measures in the frontend application of DafnckMachine v3.1, covering authentication, authorization, data protection, and security best practices.

## Security Architecture

### 1. Defense in Depth
- **Multiple Security Layers**: Layered security approach
- **Client-Side Validation**: Input validation and sanitization
- **Secure Communication**: HTTPS and secure protocols
- **Content Security**: CSP and security headers

### 2. Zero Trust Model
- **Verify Everything**: No implicit trust assumptions
- **Least Privilege**: Minimal required permissions
- **Continuous Validation**: Ongoing security checks
- **Secure by Default**: Security-first configuration

## Authentication Implementation

### JWT Token Management

```typescript
// src/auth/tokenManager.ts
interface TokenPayload {
  sub: string;
  exp: number;
  iat: number;
  roles: string[];
  permissions: string[];
}

export class TokenManager {
  private static readonly ACCESS_TOKEN_KEY = 'access_token';
  private static readonly REFRESH_TOKEN_KEY = 'refresh_token';
  private static readonly TOKEN_EXPIRY_BUFFER = 5 * 60 * 1000; // 5 minutes

  static setTokens(accessToken: string, refreshToken: string): void {
    // Use secure storage for tokens
    this.setSecureItem(this.ACCESS_TOKEN_KEY, accessToken);
    this.setSecureItem(this.REFRESH_TOKEN_KEY, refreshToken);
  }

  static getAccessToken(): string | null {
    return this.getSecureItem(this.ACCESS_TOKEN_KEY);
  }

  static getRefreshToken(): string | null {
    return this.getSecureItem(this.REFRESH_TOKEN_KEY);
  }

  static clearTokens(): void {
    this.removeSecureItem(this.ACCESS_TOKEN_KEY);
    this.removeSecureItem(this.REFRESH_TOKEN_KEY);
  }

  static isTokenValid(token: string): boolean {
    try {
      const payload = this.decodeToken(token);
      const now = Date.now();
      const expiry = payload.exp * 1000;
      
      return expiry > now + this.TOKEN_EXPIRY_BUFFER;
    } catch {
      return false;
    }
  }

  static decodeToken(token: string): TokenPayload {
    const base64Url = token.split('.')[1];
    const base64 = base64Url.replace(/-/g, '+').replace(/_/g, '/');
    const jsonPayload = decodeURIComponent(
      atob(base64)
        .split('')
        .map(c => '%' + ('00' + c.charCodeAt(0).toString(16)).slice(-2))
        .join('')
    );
    
    return JSON.parse(jsonPayload);
  }

  private static setSecureItem(key: string, value: string): void {
    // Use secure storage with encryption
    const encrypted = this.encrypt(value);
    sessionStorage.setItem(key, encrypted);
  }

  private static getSecureItem(key: string): string | null {
    const encrypted = sessionStorage.getItem(key);
    if (!encrypted) return null;
    
    try {
      return this.decrypt(encrypted);
    } catch {
      return null;
    }
  }

  private static removeSecureItem(key: string): void {
    sessionStorage.removeItem(key);
  }

  private static encrypt(text: string): string {
    // Implement client-side encryption
    // Note: This is a simplified example
    return btoa(text);
  }

  private static decrypt(encryptedText: string): string {
    // Implement client-side decryption
    return atob(encryptedText);
  }
}
```

### Secure Authentication Hook

```typescript
// src/hooks/useAuth.ts
import { useState, useEffect, useContext, createContext } from 'react';
import { TokenManager } from '../auth/tokenManager';

interface User {
  id: string;
  email: string;
  roles: string[];
  permissions: string[];
}

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
    initializeAuth();
  }, []);

  const initializeAuth = async () => {
    try {
      const token = TokenManager.getAccessToken();
      if (token && TokenManager.isTokenValid(token)) {
        const payload = TokenManager.decodeToken(token);
        await loadUserProfile(payload.sub);
      }
    } catch (error) {
      console.error('Auth initialization failed:', error);
      TokenManager.clearTokens();
    } finally {
      setIsLoading(false);
    }
  };

  const login = async (email: string, password: string) => {
    try {
      const response = await fetch('/api/auth/login', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({ email, password }),
      });

      if (!response.ok) {
        throw new Error('Login failed');
      }

      const { accessToken, refreshToken, user } = await response.json();
      
      TokenManager.setTokens(accessToken, refreshToken);
      setUser(user);
    } catch (error) {
      console.error('Login error:', error);
      throw error;
    }
  };

  const logout = () => {
    TokenManager.clearTokens();
    setUser(null);
    
    // Clear any sensitive data from memory
    window.location.href = '/login';
  };

  const refreshToken = async () => {
    try {
      const refreshToken = TokenManager.getRefreshToken();
      if (!refreshToken) {
        throw new Error('No refresh token available');
      }

      const response = await fetch('/api/auth/refresh', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({ refreshToken }),
      });

      if (!response.ok) {
        throw new Error('Token refresh failed');
      }

      const { accessToken, refreshToken: newRefreshToken } = await response.json();
      TokenManager.setTokens(accessToken, newRefreshToken);
    } catch (error) {
      console.error('Token refresh error:', error);
      logout();
    }
  };

  const loadUserProfile = async (userId: string) => {
    try {
      const response = await fetch(`/api/users/${userId}`, {
        headers: {
          'Authorization': `Bearer ${TokenManager.getAccessToken()}`,
        },
      });

      if (response.ok) {
        const userData = await response.json();
        setUser(userData);
      }
    } catch (error) {
      console.error('Failed to load user profile:', error);
    }
  };

  const value = {
    user,
    isAuthenticated: !!user,
    isLoading,
    login,
    logout,
    refreshToken,
  };

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
};

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (context === undefined) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
};
```

## Authorization & Access Control

### Role-Based Access Control (RBAC)

```typescript
// src/auth/rbac.ts
export enum Role {
  ADMIN = 'admin',
  USER = 'user',
  MODERATOR = 'moderator',
  VIEWER = 'viewer'
}

export enum Permission {
  READ_USERS = 'read:users',
  WRITE_USERS = 'write:users',
  DELETE_USERS = 'delete:users',
  READ_TASKS = 'read:tasks',
  WRITE_TASKS = 'write:tasks',
  DELETE_TASKS = 'delete:tasks',
  ADMIN_PANEL = 'access:admin'
}

const ROLE_PERMISSIONS: Record<Role, Permission[]> = {
  [Role.ADMIN]: [
    Permission.READ_USERS,
    Permission.WRITE_USERS,
    Permission.DELETE_USERS,
    Permission.READ_TASKS,
    Permission.WRITE_TASKS,
    Permission.DELETE_TASKS,
    Permission.ADMIN_PANEL
  ],
  [Role.MODERATOR]: [
    Permission.READ_USERS,
    Permission.WRITE_USERS,
    Permission.READ_TASKS,
    Permission.WRITE_TASKS
  ],
  [Role.USER]: [
    Permission.READ_TASKS,
    Permission.WRITE_TASKS
  ],
  [Role.VIEWER]: [
    Permission.READ_TASKS
  ]
};

export class AuthorizationService {
  static hasPermission(userRoles: Role[], requiredPermission: Permission): boolean {
    return userRoles.some(role => 
      ROLE_PERMISSIONS[role]?.includes(requiredPermission)
    );
  }

  static hasRole(userRoles: Role[], requiredRole: Role): boolean {
    return userRoles.includes(requiredRole);
  }

  static hasAnyRole(userRoles: Role[], requiredRoles: Role[]): boolean {
    return requiredRoles.some(role => userRoles.includes(role));
  }

  static canAccessResource(userRoles: Role[], resourcePermissions: Permission[]): boolean {
    return resourcePermissions.every(permission =>
      this.hasPermission(userRoles, permission)
    );
  }
}
```

### Protected Route Component

```tsx
// src/components/ProtectedRoute.tsx
import React from 'react';
import { Navigate, useLocation } from 'react-router-dom';
import { useAuth } from '../hooks/useAuth';
import { AuthorizationService, Role, Permission } from '../auth/rbac';

interface ProtectedRouteProps {
  children: React.ReactNode;
  requiredRoles?: Role[];
  requiredPermissions?: Permission[];
  fallbackPath?: string;
}

export const ProtectedRoute: React.FC<ProtectedRouteProps> = ({
  children,
  requiredRoles = [],
  requiredPermissions = [],
  fallbackPath = '/login'
}) => {
  const { isAuthenticated, user, isLoading } = useAuth();
  const location = useLocation();

  if (isLoading) {
    return <div className="loading-spinner">Loading...</div>;
  }

  if (!isAuthenticated) {
    return <Navigate to={fallbackPath} state={{ from: location }} replace />;
  }

  if (requiredRoles.length > 0 && user) {
    const hasRequiredRole = AuthorizationService.hasAnyRole(user.roles as Role[], requiredRoles);
    if (!hasRequiredRole) {
      return <Navigate to="/unauthorized" replace />;
    }
  }

  if (requiredPermissions.length > 0 && user) {
    const hasRequiredPermissions = requiredPermissions.every(permission =>
      AuthorizationService.hasPermission(user.roles as Role[], permission)
    );
    if (!hasRequiredPermissions) {
      return <Navigate to="/unauthorized" replace />;
    }
  }

  return <>{children}</>;
};
```

## Input Validation & Sanitization

### Input Validation Utilities

```typescript
// src/utils/validation.ts
export class InputValidator {
  private static readonly EMAIL_REGEX = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  private static readonly PHONE_REGEX = /^\+?[\d\s\-\(\)]+$/;
  private static readonly SQL_INJECTION_PATTERNS = [
    /(\b(SELECT|INSERT|UPDATE|DELETE|DROP|CREATE|ALTER|EXEC|UNION)\b)/i,
    /(--|\/\*|\*\/|;)/,
    /(\b(OR|AND)\b.*=.*)/i
  ];

  static sanitizeString(input: string): string {
    return input
      .replace(/[<>]/g, '') // Remove potential HTML tags
      .replace(/['"]/g, '') // Remove quotes
      .trim();
  }

  static sanitizeHTML(input: string): string {
    const div = document.createElement('div');
    div.textContent = input;
    return div.innerHTML;
  }

  static validateEmail(email: string): boolean {
    return this.EMAIL_REGEX.test(email) && email.length <= 254;
  }

  static validatePassword(password: string): {
    isValid: boolean;
    errors: string[];
  } {
    const errors: string[] = [];
    
    if (password.length < 8) {
      errors.push('Password must be at least 8 characters long');
    }
    
    if (!/[A-Z]/.test(password)) {
      errors.push('Password must contain at least one uppercase letter');
    }
    
    if (!/[a-z]/.test(password)) {
      errors.push('Password must contain at least one lowercase letter');
    }
    
    if (!/\d/.test(password)) {
      errors.push('Password must contain at least one number');
    }
    
    if (!/[!@#$%^&*(),.?":{}|<>]/.test(password)) {
      errors.push('Password must contain at least one special character');
    }

    return {
      isValid: errors.length === 0,
      errors
    };
  }

  static detectSQLInjection(input: string): boolean {
    return this.SQL_INJECTION_PATTERNS.some(pattern => pattern.test(input));
  }

  static validateURL(url: string): boolean {
    try {
      const urlObj = new URL(url);
      return ['http:', 'https:'].includes(urlObj.protocol);
    } catch {
      return false;
    }
  }

  static validateFileUpload(file: File, allowedTypes: string[], maxSize: number): {
    isValid: boolean;
    errors: string[];
  } {
    const errors: string[] = [];

    if (!allowedTypes.includes(file.type)) {
      errors.push(`File type ${file.type} is not allowed`);
    }

    if (file.size > maxSize) {
      errors.push(`File size exceeds maximum allowed size of ${maxSize} bytes`);
    }

    return {
      isValid: errors.length === 0,
      errors
    };
  }
}
```

### Secure Form Component

```tsx
// src/components/SecureForm.tsx
import React, { useState } from 'react';
import { InputValidator } from '../utils/validation';

interface SecureFormProps {
  onSubmit: (data: Record<string, any>) => void;
  children: React.ReactNode;
  validateOnChange?: boolean;
}

export const SecureForm: React.FC<SecureFormProps> = ({
  onSubmit,
  children,
  validateOnChange = true
}) => {
  const [formData, setFormData] = useState<Record<string, any>>({});
  const [errors, setErrors] = useState<Record<string, string[]>>({});

  const handleInputChange = (name: string, value: any) => {
    const sanitizedValue = typeof value === 'string' 
      ? InputValidator.sanitizeString(value)
      : value;

    setFormData(prev => ({
      ...prev,
      [name]: sanitizedValue
    }));

    if (validateOnChange) {
      validateField(name, sanitizedValue);
    }
  };

  const validateField = (name: string, value: any) => {
    const fieldErrors: string[] = [];

    // SQL Injection detection
    if (typeof value === 'string' && InputValidator.detectSQLInjection(value)) {
      fieldErrors.push('Invalid input detected');
    }

    // Field-specific validation
    switch (name) {
      case 'email':
        if (!InputValidator.validateEmail(value)) {
          fieldErrors.push('Invalid email format');
        }
        break;
      case 'password':
        const passwordValidation = InputValidator.validatePassword(value);
        if (!passwordValidation.isValid) {
          fieldErrors.push(...passwordValidation.errors);
        }
        break;
      case 'url':
        if (value && !InputValidator.validateURL(value)) {
          fieldErrors.push('Invalid URL format');
        }
        break;
    }

    setErrors(prev => ({
      ...prev,
      [name]: fieldErrors
    }));
  };

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();

    // Validate all fields
    const allErrors: Record<string, string[]> = {};
    Object.entries(formData).forEach(([name, value]) => {
      validateField(name, value);
    });

    const hasErrors = Object.values(allErrors).some(errors => errors.length > 0);
    if (!hasErrors) {
      onSubmit(formData);
    }
  };

  return (
    <form onSubmit={handleSubmit} noValidate>
      {React.Children.map(children, child => {
        if (React.isValidElement(child)) {
          return React.cloneElement(child, {
            onChange: handleInputChange,
            errors: errors,
            values: formData
          });
        }
        return child;
      })}
    </form>
  );
};
```

## Content Security Policy (CSP)

### CSP Configuration

```typescript
// src/security/csp.ts
export const CSP_DIRECTIVES = {
  'default-src': ["'self'"],
  'script-src': [
    "'self'",
    "'unsafe-inline'", // Only for development
    'https://apis.google.com',
    'https://www.google-analytics.com'
  ],
  'style-src': [
    "'self'",
    "'unsafe-inline'",
    'https://fonts.googleapis.com'
  ],
  'font-src': [
    "'self'",
    'https://fonts.gstatic.com'
  ],
  'img-src': [
    "'self'",
    'data:',
    'https:'
  ],
  'connect-src': [
    "'self'",
    'https://api.dafnckmachine.com',
    'wss://api.dafnckmachine.com'
  ],
  'frame-src': ["'none'"],
  'object-src': ["'none'"],
  'base-uri': ["'self'"],
  'form-action': ["'self'"],
  'frame-ancestors': ["'none'"],
  'upgrade-insecure-requests': []
};

export const generateCSPHeader = (): string => {
  return Object.entries(CSP_DIRECTIVES)
    .map(([directive, sources]) => {
      if (sources.length === 0) {
        return directive;
      }
      return `${directive} ${sources.join(' ')}`;
    })
    .join('; ');
};

// Apply CSP via meta tag (fallback)
export const applyCSPMetaTag = (): void => {
  const meta = document.createElement('meta');
  meta.httpEquiv = 'Content-Security-Policy';
  meta.content = generateCSPHeader();
  document.head.appendChild(meta);
};
```

## Secure HTTP Client

### Axios Security Configuration

```typescript
// src/api/secureClient.ts
import axios, { AxiosInstance, AxiosRequestConfig, AxiosResponse } from 'axios';
import { TokenManager } from '../auth/tokenManager';

class SecureHTTPClient {
  private client: AxiosInstance;
  private readonly baseURL: string;

  constructor(baseURL: string) {
    this.baseURL = baseURL;
    this.client = this.createClient();
    this.setupInterceptors();
  }

  private createClient(): AxiosInstance {
    return axios.create({
      baseURL: this.baseURL,
      timeout: 10000,
      withCredentials: false, // Prevent CSRF attacks
      headers: {
        'Content-Type': 'application/json',
        'X-Requested-With': 'XMLHttpRequest'
      }
    });
  }

  private setupInterceptors(): void {
    // Request interceptor
    this.client.interceptors.request.use(
      (config) => {
        // Add authentication token
        const token = TokenManager.getAccessToken();
        if (token && TokenManager.isTokenValid(token)) {
          config.headers.Authorization = `Bearer ${token}`;
        }

        // Add CSRF protection
        const csrfToken = this.getCSRFToken();
        if (csrfToken) {
          config.headers['X-CSRF-Token'] = csrfToken;
        }

        // Sanitize request data
        if (config.data) {
          config.data = this.sanitizeRequestData(config.data);
        }

        return config;
      },
      (error) => Promise.reject(error)
    );

    // Response interceptor
    this.client.interceptors.response.use(
      (response) => {
        // Validate response structure
        if (!this.isValidResponse(response)) {
          throw new Error('Invalid response structure');
        }
        return response;
      },
      async (error) => {
        if (error.response?.status === 401) {
          // Token expired, try to refresh
          try {
            await TokenManager.refreshToken();
            // Retry original request
            return this.client.request(error.config);
          } catch (refreshError) {
            // Refresh failed, redirect to login
            TokenManager.clearTokens();
            window.location.href = '/login';
          }
        }
        return Promise.reject(error);
      }
    );
  }

  private sanitizeRequestData(data: any): any {
    if (typeof data === 'string') {
      return InputValidator.sanitizeString(data);
    }
    
    if (Array.isArray(data)) {
      return data.map(item => this.sanitizeRequestData(item));
    }
    
    if (typeof data === 'object' && data !== null) {
      const sanitized: any = {};
      Object.keys(data).forEach(key => {
        sanitized[key] = this.sanitizeRequestData(data[key]);
      });
      return sanitized;
    }
    
    return data;
  }

  private isValidResponse(response: AxiosResponse): boolean {
    // Validate response has expected structure
    return response.status >= 200 && response.status < 300;
  }

  private getCSRFToken(): string | null {
    const meta = document.querySelector('meta[name="csrf-token"]');
    return meta ? meta.getAttribute('content') : null;
  }

  // Public API methods
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

  async delete<T>(url: string, config?: AxiosRequestConfig): Promise<T> {
    const response = await this.client.delete<T>(url, config);
    return response.data;
  }
}

export const secureClient = new SecureHTTPClient(process.env.REACT_APP_API_URL || '');
```

## XSS Protection

### XSS Prevention Utilities

```typescript
// src/security/xssProtection.ts
export class XSSProtection {
  private static readonly DANGEROUS_TAGS = [
    'script', 'iframe', 'object', 'embed', 'form', 'input', 'textarea', 'select'
  ];

  private static readonly DANGEROUS_ATTRIBUTES = [
    'onclick', 'onload', 'onerror', 'onmouseover', 'onfocus', 'onblur'
  ];

  static sanitizeHTML(html: string): string {
    const parser = new DOMParser();
    const doc = parser.parseFromString(html, 'text/html');
    
    // Remove dangerous tags
    this.DANGEROUS_TAGS.forEach(tag => {
      const elements = doc.querySelectorAll(tag);
      elements.forEach(el => el.remove());
    });

    // Remove dangerous attributes
    const allElements = doc.querySelectorAll('*');
    allElements.forEach(el => {
      this.DANGEROUS_ATTRIBUTES.forEach(attr => {
        if (el.hasAttribute(attr)) {
          el.removeAttribute(attr);
        }
      });

      // Remove javascript: URLs
      ['href', 'src', 'action'].forEach(attr => {
        const value = el.getAttribute(attr);
        if (value && value.toLowerCase().startsWith('javascript:')) {
          el.removeAttribute(attr);
        }
      });
    });

    return doc.body.innerHTML;
  }

  static escapeHTML(text: string): string {
    const div = document.createElement('div');
    div.textContent = text;
    return div.innerHTML;
  }

  static validateURL(url: string): boolean {
    try {
      const urlObj = new URL(url);
      return ['http:', 'https:', 'mailto:'].includes(urlObj.protocol);
    } catch {
      return false;
    }
  }
}
```

### Safe HTML Renderer Component

```tsx
// src/components/SafeHTML.tsx
import React from 'react';
import { XSSProtection } from '../security/xssProtection';

interface SafeHTMLProps {
  html: string;
  className?: string;
  allowedTags?: string[];
}

export const SafeHTML: React.FC<SafeHTMLProps> = ({
  html,
  className,
  allowedTags = ['p', 'br', 'strong', 'em', 'ul', 'ol', 'li']
}) => {
  const sanitizedHTML = React.useMemo(() => {
    return XSSProtection.sanitizeHTML(html);
  }, [html]);

  return (
    <div
      className={className}
      dangerouslySetInnerHTML={{ __html: sanitizedHTML }}
    />
  );
};
```

## Security Headers

### Security Headers Implementation

```typescript
// src/security/headers.ts
export const SECURITY_HEADERS = {
  'Strict-Transport-Security': 'max-age=31536000; includeSubDomains; preload',
  'X-Content-Type-Options': 'nosniff',
  'X-Frame-Options': 'DENY',
  'X-XSS-Protection': '1; mode=block',
  'Referrer-Policy': 'strict-origin-when-cross-origin',
  'Permissions-Policy': 'camera=(), microphone=(), geolocation=()',
  'Cross-Origin-Embedder-Policy': 'require-corp',
  'Cross-Origin-Opener-Policy': 'same-origin',
  'Cross-Origin-Resource-Policy': 'same-origin'
};

export const applySecurityHeaders = (): void => {
  // Note: These should ideally be set by the server
  // This is a client-side fallback for development
  Object.entries(SECURITY_HEADERS).forEach(([header, value]) => {
    const meta = document.createElement('meta');
    meta.httpEquiv = header;
    meta.content = value;
    document.head.appendChild(meta);
  });
};
```

## File Upload Security

### Secure File Upload Component

```tsx
// src/components/SecureFileUpload.tsx
import React, { useState } from 'react';
import { InputValidator } from '../utils/validation';

interface SecureFileUploadProps {
  onFileSelect: (file: File) => void;
  allowedTypes: string[];
  maxSize: number;
  multiple?: boolean;
}

export const SecureFileUpload: React.FC<SecureFileUploadProps> = ({
  onFileSelect,
  allowedTypes,
  maxSize,
  multiple = false
}) => {
  const [errors, setErrors] = useState<string[]>([]);

  const handleFileChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    const files = event.target.files;
    if (!files) return;

    const fileArray = Array.from(files);
    const validationErrors: string[] = [];

    fileArray.forEach(file => {
      const validation = InputValidator.validateFileUpload(file, allowedTypes, maxSize);
      if (!validation.isValid) {
        validationErrors.push(...validation.errors);
      } else {
        // Additional security checks
        if (this.isExecutableFile(file)) {
          validationErrors.push('Executable files are not allowed');
        }
        
        if (this.hasSuspiciousContent(file)) {
          validationErrors.push('File contains suspicious content');
        }
      }
    });

    setErrors(validationErrors);

    if (validationErrors.length === 0) {
      fileArray.forEach(onFileSelect);
    }
  };

  private isExecutableFile(file: File): boolean {
    const executableExtensions = ['.exe', '.bat', '.cmd', '.scr', '.pif', '.com'];
    return executableExtensions.some(ext => 
      file.name.toLowerCase().endsWith(ext)
    );
  }

  private hasSuspiciousContent(file: File): boolean {
    // Check for suspicious file signatures
    const reader = new FileReader();
    reader.onload = (e) => {
      const content = e.target?.result as string;
      const suspiciousPatterns = [
        /<script/i,
        /javascript:/i,
        /vbscript:/i,
        /onload=/i,
        /onerror=/i
      ];
      
      return suspiciousPatterns.some(pattern => pattern.test(content));
    };
    
    return false; // Simplified for example
  }

  return (
    <div className="secure-file-upload">
      <input
        type="file"
        onChange={handleFileChange}
        accept={allowedTypes.join(',')}
        multiple={multiple}
      />
      {errors.length > 0 && (
        <div className="error-messages">
          {errors.map((error, index) => (
            <div key={index} className="error-message">
              {error}
            </div>
          ))}
        </div>
      )}
    </div>
  );
};
```

## Security Testing

### Security Test Suite

```typescript
// src/__tests__/security.test.ts
import { InputValidator } from '../utils/validation';
import { XSSProtection } from '../security/xssProtection';
import { TokenManager } from '../auth/tokenManager';

describe('Security Features', () => {
  describe('Input Validation', () => {
    it('should detect SQL injection attempts', () => {
      const maliciousInputs = [
        "'; DROP TABLE users; --",
        "1' OR '1'='1",
        "UNION SELECT * FROM users"
      ];

      maliciousInputs.forEach(input => {
        expect(InputValidator.detectSQLInjection(input)).toBe(true);
      });
    });

    it('should validate email addresses correctly', () => {
      expect(InputValidator.validateEmail('test@example.com')).toBe(true);
      expect(InputValidator.validateEmail('invalid-email')).toBe(false);
      expect(InputValidator.validateEmail('test@')).toBe(false);
    });

    it('should enforce strong password requirements', () => {
      const weakPassword = 'password';
      const strongPassword = 'StrongP@ssw0rd123';

      expect(InputValidator.validatePassword(weakPassword).isValid).toBe(false);
      expect(InputValidator.validatePassword(strongPassword).isValid).toBe(true);
    });
  });

  describe('XSS Protection', () => {
    it('should sanitize malicious HTML', () => {
      const maliciousHTML = '<script>alert("XSS")</script><p>Safe content</p>';
      const sanitized = XSSProtection.sanitizeHTML(maliciousHTML);
      
      expect(sanitized).not.toContain('<script>');
      expect(sanitized).toContain('<p>Safe content</p>');
    });

    it('should remove dangerous attributes', () => {
      const maliciousHTML = '<div onclick="alert(\'XSS\')">Click me</div>';
      const sanitized = XSSProtection.sanitizeHTML(maliciousHTML);
      
      expect(sanitized).not.toContain('onclick');
    });
  });

  describe('Token Management', () => {
    it('should validate token expiry correctly', () => {
      const expiredToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyLCJleHAiOjE1MTYyMzkwMjJ9.invalid';
      
      expect(TokenManager.isTokenValid(expiredToken)).toBe(false);
    });

    it('should clear tokens on logout', () => {
      TokenManager.setTokens('access-token', 'refresh-token');
      TokenManager.clearTokens();
      
      expect(TokenManager.getAccessToken()).toBeNull();
      expect(TokenManager.getRefreshToken()).toBeNull();
    });
  });
});
```

## Security Monitoring

### Security Event Logging

```typescript
// src/security/monitoring.ts
interface SecurityEvent {
  type: 'auth_failure' | 'xss_attempt' | 'sql_injection' | 'unauthorized_access';
  timestamp: number;
  userAgent: string;
  ip?: string;
  details: Record<string, any>;
}

export class SecurityMonitor {
  private static events: SecurityEvent[] = [];

  static logSecurityEvent(event: Omit<SecurityEvent, 'timestamp' | 'userAgent'>): void {
    const securityEvent: SecurityEvent = {
      ...event,
      timestamp: Date.now(),
      userAgent: navigator.userAgent
    };

    this.events.push(securityEvent);
    this.sendToServer(securityEvent);
  }

  static logAuthFailure(details: Record<string, any>): void {
    this.logSecurityEvent({
      type: 'auth_failure',
      details
    });
  }

  static logXSSAttempt(details: Record<string, any>): void {
    this.logSecurityEvent({
      type: 'xss_attempt',
      details
    });
  }

  static logSQLInjectionAttempt(details: Record<string, any>): void {
    this.logSecurityEvent({
      type: 'sql_injection',
      details
    });
  }

  static logUnauthorizedAccess(details: Record<string, any>): void {
    this.logSecurityEvent({
      type: 'unauthorized_access',
      details
    });
  }

  private static async sendToServer(event: SecurityEvent): Promise<void> {
    try {
      await fetch('/api/security/events', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(event)
      });
    } catch (error) {
      console.error('Failed to send security event:', error);
    }
  }

  static getEvents(): SecurityEvent[] {
    return [...this.events];
  }

  static clearEvents(): void {
    this.events = [];
  }
}
```

## Best Practices

### 1. Authentication Security
- **Strong Password Policy**: Enforce complex passwords
- **Multi-Factor Authentication**: Implement 2FA/MFA
- **Session Management**: Secure token handling and rotation
- **Account Lockout**: Prevent brute force attacks

### 2. Data Protection
- **Input Sanitization**: Validate and sanitize all inputs
- **Output Encoding**: Encode data before display
- **Secure Storage**: Encrypt sensitive data
- **Data Minimization**: Collect only necessary data

### 3. Communication Security
- **HTTPS Only**: Force secure connections
- **Certificate Pinning**: Validate server certificates
- **API Security**: Secure API endpoints and authentication
- **CORS Configuration**: Proper cross-origin settings

### 4. Client-Side Security
- **CSP Implementation**: Content Security Policy
- **XSS Prevention**: Cross-site scripting protection
- **CSRF Protection**: Cross-site request forgery prevention
- **Secure Headers**: Security-related HTTP headers

## Compliance and Auditing

### Security Audit Checklist
- [ ] **Authentication**: Secure login/logout implementation
- [ ] **Authorization**: Proper access control mechanisms
- [ ] **Input Validation**: All inputs validated and sanitized
- [ ] **XSS Protection**: Cross-site scripting prevention
- [ ] **CSRF Protection**: Cross-site request forgery prevention
- [ ] **Secure Communication**: HTTPS and secure protocols
- [ ] **Data Protection**: Sensitive data encryption
- [ ] **Security Headers**: Proper HTTP security headers
- [ ] **Error Handling**: Secure error messages
- [ ] **Logging**: Security event monitoring

This comprehensive security implementation ensures DafnckMachine v3.1 maintains the highest security standards while providing a seamless user experience. 