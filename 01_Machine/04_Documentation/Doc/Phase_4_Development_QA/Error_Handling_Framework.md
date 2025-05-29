# Error Handling Framework - DafnckMachine v3.1

## Overview
Comprehensive error handling framework for the DafnckMachine v3.1 project, covering error classification, handling strategies, logging, monitoring, and recovery mechanisms.

## Error Classification System

### Error Categories

```typescript
// Error Classification Hierarchy
export enum ErrorCategory {
  // System-level errors
  SYSTEM = 'system',
  DATABASE = 'database',
  NETWORK = 'network',
  EXTERNAL_SERVICE = 'external_service',
  
  // Application-level errors
  VALIDATION = 'validation',
  AUTHENTICATION = 'authentication',
  AUTHORIZATION = 'authorization',
  BUSINESS_LOGIC = 'business_logic',
  
  // User-level errors
  USER_INPUT = 'user_input',
  NOT_FOUND = 'not_found',
  CONFLICT = 'conflict',
  RATE_LIMIT = 'rate_limit'
}

export enum ErrorSeverity {
  LOW = 'low',
  MEDIUM = 'medium',
  HIGH = 'high',
  CRITICAL = 'critical'
}

export enum ErrorRecovery {
  RETRY = 'retry',
  FALLBACK = 'fallback',
  CIRCUIT_BREAKER = 'circuit_breaker',
  MANUAL = 'manual',
  NONE = 'none'
}
```

### Base Error Classes

```typescript
// Base Application Error
export abstract class AppError extends Error {
  public readonly category: ErrorCategory;
  public readonly severity: ErrorSeverity;
  public readonly recovery: ErrorRecovery;
  public readonly statusCode: number;
  public readonly isOperational: boolean;
  public readonly timestamp: Date;
  public readonly correlationId: string;
  public readonly context: Record<string, any>;

  constructor(
    message: string,
    category: ErrorCategory,
    severity: ErrorSeverity,
    statusCode: number,
    recovery: ErrorRecovery = ErrorRecovery.NONE,
    context: Record<string, any> = {}
  ) {
    super(message);
    
    this.name = this.constructor.name;
    this.category = category;
    this.severity = severity;
    this.recovery = recovery;
    this.statusCode = statusCode;
    this.isOperational = true;
    this.timestamp = new Date();
    this.correlationId = this.generateCorrelationId();
    this.context = context;

    Error.captureStackTrace(this, this.constructor);
  }

  private generateCorrelationId(): string {
    return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }

  toJSON(): object {
    return {
      name: this.name,
      message: this.message,
      category: this.category,
      severity: this.severity,
      recovery: this.recovery,
      statusCode: this.statusCode,
      timestamp: this.timestamp.toISOString(),
      correlationId: this.correlationId,
      context: this.context,
      stack: this.stack
    };
  }
}
```

### Specific Error Types

```typescript
// Validation Errors
export class ValidationError extends AppError {
  public readonly field?: string;
  public readonly value?: any;
  public readonly constraint?: string;

  constructor(
    message: string,
    field?: string,
    value?: any,
    constraint?: string,
    context: Record<string, any> = {}
  ) {
    super(
      message,
      ErrorCategory.VALIDATION,
      ErrorSeverity.LOW,
      400,
      ErrorRecovery.NONE,
      { field, value, constraint, ...context }
    );
    
    this.field = field;
    this.value = value;
    this.constraint = constraint;
  }
}

// Authentication Errors
export class AuthenticationError extends AppError {
  constructor(message: string = 'Authentication failed', context: Record<string, any> = {}) {
    super(
      message,
      ErrorCategory.AUTHENTICATION,
      ErrorSeverity.MEDIUM,
      401,
      ErrorRecovery.NONE,
      context
    );
  }
}

// Authorization Errors
export class AuthorizationError extends AppError {
  public readonly requiredPermission?: string;
  public readonly userRole?: string;

  constructor(
    message: string = 'Insufficient permissions',
    requiredPermission?: string,
    userRole?: string,
    context: Record<string, any> = {}
  ) {
    super(
      message,
      ErrorCategory.AUTHORIZATION,
      ErrorSeverity.MEDIUM,
      403,
      ErrorRecovery.NONE,
      { requiredPermission, userRole, ...context }
    );
    
    this.requiredPermission = requiredPermission;
    this.userRole = userRole;
  }
}

// Database Errors
export class DatabaseError extends AppError {
  public readonly operation?: string;
  public readonly table?: string;
  public readonly constraint?: string;

  constructor(
    message: string,
    operation?: string,
    table?: string,
    constraint?: string,
    context: Record<string, any> = {}
  ) {
    super(
      message,
      ErrorCategory.DATABASE,
      ErrorSeverity.HIGH,
      500,
      ErrorRecovery.RETRY,
      { operation, table, constraint, ...context }
    );
    
    this.operation = operation;
    this.table = table;
    this.constraint = constraint;
  }
}

// External Service Errors
export class ExternalServiceError extends AppError {
  public readonly service?: string;
  public readonly endpoint?: string;
  public readonly responseStatus?: number;

  constructor(
    message: string,
    service?: string,
    endpoint?: string,
    responseStatus?: number,
    context: Record<string, any> = {}
  ) {
    super(
      message,
      ErrorCategory.EXTERNAL_SERVICE,
      ErrorSeverity.HIGH,
      502,
      ErrorRecovery.CIRCUIT_BREAKER,
      { service, endpoint, responseStatus, ...context }
    );
    
    this.service = service;
    this.endpoint = endpoint;
    this.responseStatus = responseStatus;
  }
}

// Business Logic Errors
export class BusinessLogicError extends AppError {
  public readonly rule?: string;
  public readonly entity?: string;

  constructor(
    message: string,
    rule?: string,
    entity?: string,
    context: Record<string, any> = {}
  ) {
    super(
      message,
      ErrorCategory.BUSINESS_LOGIC,
      ErrorSeverity.MEDIUM,
      422,
      ErrorRecovery.NONE,
      { rule, entity, ...context }
    );
    
    this.rule = rule;
    this.entity = entity;
  }
}

// Not Found Errors
export class NotFoundError extends AppError {
  public readonly resource?: string;
  public readonly identifier?: string;

  constructor(
    message: string = 'Resource not found',
    resource?: string,
    identifier?: string,
    context: Record<string, any> = {}
  ) {
    super(
      message,
      ErrorCategory.NOT_FOUND,
      ErrorSeverity.LOW,
      404,
      ErrorRecovery.NONE,
      { resource, identifier, ...context }
    );
    
    this.resource = resource;
    this.identifier = identifier;
  }
}

// Rate Limit Errors
export class RateLimitError extends AppError {
  public readonly limit?: number;
  public readonly window?: number;
  public readonly retryAfter?: number;

  constructor(
    message: string = 'Rate limit exceeded',
    limit?: number,
    window?: number,
    retryAfter?: number,
    context: Record<string, any> = {}
  ) {
    super(
      message,
      ErrorCategory.RATE_LIMIT,
      ErrorSeverity.MEDIUM,
      429,
      ErrorRecovery.RETRY,
      { limit, window, retryAfter, ...context }
    );
    
    this.limit = limit;
    this.window = window;
    this.retryAfter = retryAfter;
  }
}
```

## Error Handler Middleware

### Global Error Handler

```typescript
// Global Error Handling Middleware
export class ErrorHandler {
  private readonly logger: Logger;
  private readonly notificationService: NotificationService;

  constructor(logger: Logger, notificationService: NotificationService) {
    this.logger = logger;
    this.notificationService = notificationService;
  }

  handle() {
    return (error: Error, req: Request, res: Response, next: NextFunction) => {
      // Ensure error is an AppError
      const appError = this.normalizeError(error);
      
      // Log the error
      this.logError(appError, req);
      
      // Send notifications for critical errors
      if (appError.severity === ErrorSeverity.CRITICAL) {
        this.notifyTeam(appError, req);
      }
      
      // Send response to client
      this.sendErrorResponse(appError, res);
    };
  }

  private normalizeError(error: Error): AppError {
    if (error instanceof AppError) {
      return error;
    }

    // Handle known error types
    if (error.name === 'ValidationError') {
      return new ValidationError(error.message);
    }

    if (error.name === 'CastError') {
      return new ValidationError('Invalid data format');
    }

    if (error.name === 'MongoError' || error.name === 'MongooseError') {
      return new DatabaseError(error.message);
    }

    if (error.name === 'JsonWebTokenError') {
      return new AuthenticationError('Invalid token');
    }

    if (error.name === 'TokenExpiredError') {
      return new AuthenticationError('Token expired');
    }

    // Default to system error
    return new SystemError(
      error.message || 'An unexpected error occurred',
      { originalError: error.name }
    );
  }

  private logError(error: AppError, req: Request): void {
    const logData = {
      error: error.toJSON(),
      request: {
        method: req.method,
        url: req.url,
        headers: this.sanitizeHeaders(req.headers),
        body: this.sanitizeBody(req.body),
        user: req.user?.id,
        ip: req.ip,
        userAgent: req.get('User-Agent')
      }
    };

    switch (error.severity) {
      case ErrorSeverity.CRITICAL:
        this.logger.error('Critical error occurred', logData);
        break;
      case ErrorSeverity.HIGH:
        this.logger.error('High severity error', logData);
        break;
      case ErrorSeverity.MEDIUM:
        this.logger.warn('Medium severity error', logData);
        break;
      case ErrorSeverity.LOW:
        this.logger.info('Low severity error', logData);
        break;
    }
  }

  private async notifyTeam(error: AppError, req: Request): Promise<void> {
    try {
      await this.notificationService.sendAlert({
        title: `Critical Error: ${error.name}`,
        message: error.message,
        severity: error.severity,
        correlationId: error.correlationId,
        url: req.url,
        timestamp: error.timestamp
      });
    } catch (notificationError) {
      this.logger.error('Failed to send error notification', {
        originalError: error.toJSON(),
        notificationError: notificationError.message
      });
    }
  }

  private sendErrorResponse(error: AppError, res: Response): void {
    const isDevelopment = process.env.NODE_ENV === 'development';
    
    const errorResponse: any = {
      error: {
        message: error.message,
        category: error.category,
        correlationId: error.correlationId,
        timestamp: error.timestamp.toISOString()
      }
    };

    // Add additional context in development
    if (isDevelopment) {
      errorResponse.error.stack = error.stack;
      errorResponse.error.context = error.context;
    }

    // Add retry information for retryable errors
    if (error.recovery === ErrorRecovery.RETRY && error instanceof RateLimitError) {
      errorResponse.retryAfter = error.retryAfter;
    }

    res.status(error.statusCode).json(errorResponse);
  }

  private sanitizeHeaders(headers: any): any {
    const sanitized = { ...headers };
    delete sanitized.authorization;
    delete sanitized.cookie;
    return sanitized;
  }

  private sanitizeBody(body: any): any {
    if (!body) return body;
    
    const sanitized = { ...body };
    delete sanitized.password;
    delete sanitized.token;
    delete sanitized.secret;
    return sanitized;
  }
}
```

### Async Error Handler

```typescript
// Async Error Wrapper
export function asyncHandler(fn: Function) {
  return (req: Request, res: Response, next: NextFunction) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
}

// Usage example
export const getUserById = asyncHandler(async (req: Request, res: Response) => {
  const { id } = req.params;
  
  const user = await userService.findById(id);
  if (!user) {
    throw new NotFoundError('User not found', 'user', id);
  }
  
  res.json({ user });
});
```

## Error Recovery Strategies

### Retry Mechanism

```typescript
// Retry Configuration
interface RetryConfig {
  maxAttempts: number;
  baseDelay: number;
  maxDelay: number;
  backoffMultiplier: number;
  retryableErrors: ErrorCategory[];
}

export class RetryHandler {
  private readonly config: RetryConfig;

  constructor(config: RetryConfig) {
    this.config = config;
  }

  async execute<T>(
    operation: () => Promise<T>,
    context: string = 'unknown'
  ): Promise<T> {
    let lastError: Error;
    
    for (let attempt = 1; attempt <= this.config.maxAttempts; attempt++) {
      try {
        return await operation();
      } catch (error) {
        lastError = error;
        
        // Check if error is retryable
        if (!this.isRetryable(error) || attempt === this.config.maxAttempts) {
          throw error;
        }
        
        // Calculate delay
        const delay = this.calculateDelay(attempt);
        
        logger.warn(`Retry attempt ${attempt}/${this.config.maxAttempts} for ${context}`, {
          error: error.message,
          delay,
          attempt
        });
        
        // Wait before retry
        await this.sleep(delay);
      }
    }
    
    throw lastError!;
  }

  private isRetryable(error: Error): boolean {
    if (error instanceof AppError) {
      return this.config.retryableErrors.includes(error.category) &&
             error.recovery === ErrorRecovery.RETRY;
    }
    
    // Default retry logic for non-AppErrors
    return error.name === 'TimeoutError' ||
           error.name === 'ConnectionError' ||
           error.message.includes('ECONNRESET');
  }

  private calculateDelay(attempt: number): number {
    const delay = this.config.baseDelay * Math.pow(this.config.backoffMultiplier, attempt - 1);
    return Math.min(delay, this.config.maxDelay);
  }

  private sleep(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

// Default retry configuration
export const defaultRetryConfig: RetryConfig = {
  maxAttempts: 3,
  baseDelay: 1000, // 1 second
  maxDelay: 10000, // 10 seconds
  backoffMultiplier: 2,
  retryableErrors: [
    ErrorCategory.DATABASE,
    ErrorCategory.NETWORK,
    ErrorCategory.EXTERNAL_SERVICE,
    ErrorCategory.RATE_LIMIT
  ]
};
```

### Circuit Breaker Pattern

```typescript
// Circuit Breaker Implementation
export enum CircuitState {
  CLOSED = 'closed',
  OPEN = 'open',
  HALF_OPEN = 'half_open'
}

interface CircuitBreakerConfig {
  failureThreshold: number;
  recoveryTimeout: number;
  monitoringPeriod: number;
  halfOpenMaxCalls: number;
}

export class CircuitBreaker {
  private state: CircuitState = CircuitState.CLOSED;
  private failureCount: number = 0;
  private lastFailureTime: number = 0;
  private halfOpenCalls: number = 0;
  private readonly config: CircuitBreakerConfig;
  private readonly name: string;

  constructor(name: string, config: CircuitBreakerConfig) {
    this.name = name;
    this.config = config;
  }

  async execute<T>(operation: () => Promise<T>): Promise<T> {
    if (this.state === CircuitState.OPEN) {
      if (this.shouldAttemptReset()) {
        this.state = CircuitState.HALF_OPEN;
        this.halfOpenCalls = 0;
        logger.info(`Circuit breaker ${this.name} transitioning to HALF_OPEN`);
      } else {
        throw new ExternalServiceError(
          `Circuit breaker ${this.name} is OPEN`,
          this.name,
          undefined,
          503
        );
      }
    }

    try {
      const result = await operation();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  private shouldAttemptReset(): boolean {
    return Date.now() - this.lastFailureTime >= this.config.recoveryTimeout;
  }

  private onSuccess(): void {
    this.failureCount = 0;
    
    if (this.state === CircuitState.HALF_OPEN) {
      this.state = CircuitState.CLOSED;
      logger.info(`Circuit breaker ${this.name} transitioning to CLOSED`);
    }
  }

  private onFailure(): void {
    this.failureCount++;
    this.lastFailureTime = Date.now();

    if (this.state === CircuitState.HALF_OPEN) {
      this.state = CircuitState.OPEN;
      logger.warn(`Circuit breaker ${this.name} transitioning to OPEN from HALF_OPEN`);
    } else if (this.failureCount >= this.config.failureThreshold) {
      this.state = CircuitState.OPEN;
      logger.warn(`Circuit breaker ${this.name} transitioning to OPEN`, {
        failureCount: this.failureCount,
        threshold: this.config.failureThreshold
      });
    }
  }

  getState(): CircuitState {
    return this.state;
  }

  getMetrics(): object {
    return {
      name: this.name,
      state: this.state,
      failureCount: this.failureCount,
      lastFailureTime: this.lastFailureTime
    };
  }
}
```

### Fallback Mechanisms

```typescript
// Fallback Handler
export class FallbackHandler {
  private readonly fallbacks: Map<string, () => Promise<any>> = new Map();

  registerFallback(service: string, fallback: () => Promise<any>): void {
    this.fallbacks.set(service, fallback);
  }

  async executeWithFallback<T>(
    service: string,
    primaryOperation: () => Promise<T>,
    fallbackData?: T
  ): Promise<T> {
    try {
      return await primaryOperation();
    } catch (error) {
      logger.warn(`Primary operation failed for ${service}, attempting fallback`, {
        error: error.message
      });

      // Try registered fallback
      const fallback = this.fallbacks.get(service);
      if (fallback) {
        try {
          return await fallback();
        } catch (fallbackError) {
          logger.error(`Fallback failed for ${service}`, {
            primaryError: error.message,
            fallbackError: fallbackError.message
          });
        }
      }

      // Use provided fallback data
      if (fallbackData !== undefined) {
        logger.info(`Using fallback data for ${service}`);
        return fallbackData;
      }

      // No fallback available, re-throw original error
      throw error;
    }
  }
}

// Example usage
const fallbackHandler = new FallbackHandler();

// Register fallbacks
fallbackHandler.registerFallback('user-service', async () => {
  return { id: 'unknown', name: 'Guest User' };
});

fallbackHandler.registerFallback('recommendation-service', async () => {
  return []; // Empty recommendations
});
```

## Error Monitoring and Alerting

### Error Metrics Collection

```typescript
// Error Metrics Collector
export class ErrorMetrics {
  private readonly metrics: Map<string, ErrorMetric> = new Map();

  recordError(error: AppError): void {
    const key = `${error.category}:${error.name}`;
    
    if (!this.metrics.has(key)) {
      this.metrics.set(key, {
        category: error.category,
        name: error.name,
        count: 0,
        lastOccurrence: error.timestamp,
        severity: error.severity
      });
    }

    const metric = this.metrics.get(key)!;
    metric.count++;
    metric.lastOccurrence = error.timestamp;
  }

  getMetrics(): ErrorMetric[] {
    return Array.from(this.metrics.values());
  }

  getMetricsByCategory(category: ErrorCategory): ErrorMetric[] {
    return this.getMetrics().filter(metric => metric.category === category);
  }

  getTopErrors(limit: number = 10): ErrorMetric[] {
    return this.getMetrics()
      .sort((a, b) => b.count - a.count)
      .slice(0, limit);
  }

  reset(): void {
    this.metrics.clear();
  }
}

interface ErrorMetric {
  category: ErrorCategory;
  name: string;
  count: number;
  lastOccurrence: Date;
  severity: ErrorSeverity;
}
```

### Health Check Integration

```typescript
// Health Check with Error Monitoring
export class HealthCheckService {
  private readonly errorMetrics: ErrorMetrics;
  private readonly circuitBreakers: Map<string, CircuitBreaker>;

  constructor(errorMetrics: ErrorMetrics, circuitBreakers: Map<string, CircuitBreaker>) {
    this.errorMetrics = errorMetrics;
    this.circuitBreakers = circuitBreakers;
  }

  async getHealthStatus(): Promise<HealthStatus> {
    const errorMetrics = this.errorMetrics.getMetrics();
    const criticalErrors = errorMetrics.filter(m => m.severity === ErrorSeverity.CRITICAL);
    const recentErrors = errorMetrics.filter(m => 
      Date.now() - m.lastOccurrence.getTime() < 300000 // Last 5 minutes
    );

    const circuitBreakerStatus = Array.from(this.circuitBreakers.entries()).map(
      ([name, breaker]) => ({
        name,
        state: breaker.getState(),
        metrics: breaker.getMetrics()
      })
    );

    const isHealthy = criticalErrors.length === 0 && 
                     recentErrors.length < 10 &&
                     circuitBreakerStatus.every(cb => cb.state !== CircuitState.OPEN);

    return {
      status: isHealthy ? 'healthy' : 'unhealthy',
      timestamp: new Date().toISOString(),
      errors: {
        total: errorMetrics.length,
        critical: criticalErrors.length,
        recent: recentErrors.length,
        topErrors: this.errorMetrics.getTopErrors(5)
      },
      circuitBreakers: circuitBreakerStatus
    };
  }
}
```

## Error Testing

### Error Simulation for Testing

```typescript
// Error Simulation Service
export class ErrorSimulator {
  private readonly enabledInProduction = false;

  simulateError(type: string, probability: number = 1.0): void {
    if (process.env.NODE_ENV === 'production' && !this.enabledInProduction) {
      return;
    }

    if (Math.random() > probability) {
      return;
    }

    switch (type) {
      case 'database':
        throw new DatabaseError('Simulated database connection failure');
      case 'external_service':
        throw new ExternalServiceError('Simulated external service timeout');
      case 'validation':
        throw new ValidationError('Simulated validation error');
      case 'auth':
        throw new AuthenticationError('Simulated authentication failure');
      default:
        throw new Error('Simulated generic error');
    }
  }
}

// Test Cases for Error Handling
describe('Error Handling', () => {
  let errorHandler: ErrorHandler;
  let errorMetrics: ErrorMetrics;

  beforeEach(() => {
    errorMetrics = new ErrorMetrics();
    errorHandler = new ErrorHandler(logger, notificationService);
  });

  describe('Error Classification', () => {
    it('should classify validation errors correctly', () => {
      const error = new ValidationError('Invalid email format', 'email');
      
      expect(error.category).toBe(ErrorCategory.VALIDATION);
      expect(error.severity).toBe(ErrorSeverity.LOW);
      expect(error.statusCode).toBe(400);
    });

    it('should classify database errors correctly', () => {
      const error = new DatabaseError('Connection timeout', 'SELECT', 'users');
      
      expect(error.category).toBe(ErrorCategory.DATABASE);
      expect(error.severity).toBe(ErrorSeverity.HIGH);
      expect(error.recovery).toBe(ErrorRecovery.RETRY);
    });
  });

  describe('Retry Mechanism', () => {
    it('should retry retryable errors', async () => {
      const retryHandler = new RetryHandler(defaultRetryConfig);
      let attempts = 0;
      
      const operation = async () => {
        attempts++;
        if (attempts < 3) {
          throw new DatabaseError('Temporary connection issue');
        }
        return 'success';
      };

      const result = await retryHandler.execute(operation);
      
      expect(result).toBe('success');
      expect(attempts).toBe(3);
    });
  });

  describe('Circuit Breaker', () => {
    it('should open circuit after threshold failures', async () => {
      const circuitBreaker = new CircuitBreaker('test-service', {
        failureThreshold: 3,
        recoveryTimeout: 5000,
        monitoringPeriod: 10000,
        halfOpenMaxCalls: 1
      });

      const failingOperation = async () => {
        throw new ExternalServiceError('Service unavailable');
      };

      // Trigger failures to open circuit
      for (let i = 0; i < 3; i++) {
        try {
          await circuitBreaker.execute(failingOperation);
        } catch (error) {
          // Expected
        }
      }

      expect(circuitBreaker.getState()).toBe(CircuitState.OPEN);
    });
  });
});
```

## Related Documentation

- [Database Schema Design](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Database_Schema_Design.md)
- [API Documentation Complete](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/API_Documentation_Complete.md)
- [Security Implementation Guide](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Security_Implementation_Guide.md)
- [Performance Optimization Strategies](mdc:01_Machine/04_Documentation/Doc/Phase_4_Development_QA/Performance_Optimization_Strategies.md)

## Metadata

- **Last Updated**: 2024-01-15
- **Version**: 1.0.0
- **Created By**: coding-agent
- **Related Workflows**: 13_Backend_Development.md 