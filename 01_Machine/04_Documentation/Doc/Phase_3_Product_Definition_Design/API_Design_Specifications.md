# API Design Specifications
## DafnckMachine v3.1 - RESTful API Architecture & Integration Standards

### Document Overview
This document defines the comprehensive API design specifications for DafnckMachine v3.1, establishing RESTful API architecture, endpoint design patterns, authentication mechanisms, rate limiting strategies, versioning approaches, and integration standards to ensure consistent, secure, and scalable API implementation across the AI agent orchestration platform.

### API Design Philosophy

#### Core Principles
1. **RESTful Design**: Follow REST architectural principles and HTTP standards
2. **Consistency**: Uniform API patterns and response structures
3. **Security**: Robust authentication and authorization mechanisms
4. **Performance**: Optimized for speed and efficiency
5. **Scalability**: Designed to handle growing traffic and complexity
6. **Developer Experience**: Clear documentation and intuitive design
7. **Versioning**: Backward compatibility and smooth evolution

#### Design Standards
- **HTTP Methods**: Proper use of GET, POST, PUT, PATCH, DELETE
- **Status Codes**: Meaningful HTTP status codes for all responses
- **Resource Naming**: Clear, consistent, and hierarchical resource paths
- **Error Handling**: Standardized error responses with detailed information
- **Pagination**: Consistent pagination patterns for list endpoints
- **Filtering**: Flexible filtering and sorting capabilities

### API Architecture Overview

#### Base URL Structure
```
Production:  https://api.dafnckmachine.com/v1
Staging:     https://api-staging.dafnckmachine.com/v1
Development: https://api-dev.dafnckmachine.com/v1
```

#### API Versioning Strategy
```typescript
// URL-based versioning (preferred)
GET /v1/agents
GET /v2/agents

// Header-based versioning (alternative)
GET /agents
Headers: {
  "API-Version": "v1",
  "Accept": "application/vnd.dafnckmachine.v1+json"
}

// Version compatibility matrix
interface APIVersioning {
  current: "v1";
  supported: ["v1"];
  deprecated: [];
  sunset_schedule: {
    v1: null; // Current version
  };
  
  breaking_changes: {
    v2: {
      planned_date: "2025-06-01";
      migration_guide: "/docs/migration/v1-to-v2";
      deprecation_notice: "2025-03-01";
    };
  };
}
```

### Authentication & Authorization

#### Authentication Methods
```typescript
// JWT Bearer Token Authentication
interface JWTAuthentication {
  type: "Bearer";
  token: string;
  header: "Authorization: Bearer <token>";
  
  token_structure: {
    header: {
      alg: "RS256";
      typ: "JWT";
    };
    payload: {
      sub: string;        // User ID
      iss: "dafnckmachine"; // Issuer
      aud: "api.dafnckmachine.com"; // Audience
      exp: number;        // Expiration
      iat: number;        // Issued at
      roles: string[];    // User roles
      permissions: string[]; // Specific permissions
    };
  };
}

// API Key Authentication (for service-to-service)
interface APIKeyAuthentication {
  type: "API-Key";
  header: "X-API-Key: <api_key>";
  signature: "X-API-Signature: <hmac_signature>";
  
  signature_generation: {
    algorithm: "HMAC-SHA256";
    payload: "HTTP_METHOD + REQUEST_PATH + REQUEST_BODY + TIMESTAMP";
    secret: "api_secret_key";
  };
}

// OAuth 2.0 (for third-party integrations)
interface OAuth2Authentication {
  type: "OAuth2";
  flows: ["authorization_code", "client_credentials"];
  scopes: ["read", "write", "admin"];
  
  endpoints: {
    authorization: "/oauth/authorize";
    token: "/oauth/token";
    revoke: "/oauth/revoke";
    introspect: "/oauth/introspect";
  };
}
```

#### Authorization Framework
```typescript
// Role-Based Access Control (RBAC)
interface RBACSystem {
  roles: {
    admin: {
      permissions: ["*"];
      description: "Full system access";
    };
    agent_operator: {
      permissions: [
        "agents:read",
        "agents:write",
        "agents:execute",
        "workflows:read",
        "workflows:write"
      ];
      description: "Agent management and workflow execution";
    };
    viewer: {
      permissions: [
        "agents:read",
        "workflows:read",
        "reports:read"
      ];
      description: "Read-only access to system resources";
    };
  };
  
  permission_format: "resource:action";
  wildcard_support: true;
  inheritance: "hierarchical";
}

// Permission checking middleware
const checkPermission = (resource: string, action: string) => {
  return (req: AuthenticatedRequest, res: Response, next: NextFunction) => {
    const userPermissions = req.user.permissions;
    const requiredPermission = `${resource}:${action}`;
    
    if (hasPermission(userPermissions, requiredPermission)) {
      next();
    } else {
      res.status(403).json({
        error: "Forbidden",
        message: `Insufficient permissions for ${requiredPermission}`,
        required_permission: requiredPermission
      });
    }
  };
};
```

### API Endpoint Design

#### Agent Management Endpoints
```typescript
// GET /v1/agents - List agents
interface ListAgentsEndpoint {
  method: "GET";
  path: "/v1/agents";
  
  query_parameters: {
    page?: number;        // Page number (default: 1)
    limit?: number;       // Items per page (default: 20, max: 100)
    status?: "active" | "inactive" | "error" | "maintenance";
    type?: "orchestrator" | "worker" | "monitor";
    capabilities?: string; // Comma-separated list
    sort?: "name" | "created_at" | "updated_at";
    order?: "asc" | "desc";
    search?: string;      // Search in name and description
  };
  
  response: {
    success: boolean;
    data: Agent[];
    pagination: {
      page: number;
      limit: number;
      total: number;
      total_pages: number;
      has_next: boolean;
      has_prev: boolean;
    };
    meta: {
      total_active: number;
      total_inactive: number;
      response_time: number;
    };
  };
}

// POST /v1/agents - Create agent
interface CreateAgentEndpoint {
  method: "POST";
  path: "/v1/agents";
  
  request_body: {
    name: string;
    type: "orchestrator" | "worker" | "monitor";
    capabilities: string[];
    configuration: Record<string, any>;
    metadata?: Record<string, any>;
    auto_start?: boolean;
  };
  
  response: {
    success: boolean;
    data: Agent;
    message: string;
  };
  
  validation: {
    name: {
      required: true;
      min_length: 1;
      max_length: 255;
      pattern: "^[a-zA-Z0-9_-]+$";
    };
    capabilities: {
      required: true;
      min_items: 1;
      max_items: 50;
    };
  };
}

// GET /v1/agents/{id} - Get agent details
interface GetAgentEndpoint {
  method: "GET";
  path: "/v1/agents/{id}";
  
  path_parameters: {
    id: string; // Agent UUID
  };
  
  query_parameters: {
    include?: "metrics" | "logs" | "workflows"; // Additional data to include
  };
  
  response: {
    success: boolean;
    data: Agent & {
      metrics?: AgentMetrics;
      recent_logs?: LogEntry[];
      active_workflows?: Workflow[];
    };
  };
}

// PUT /v1/agents/{id} - Update agent
interface UpdateAgentEndpoint {
  method: "PUT";
  path: "/v1/agents/{id}";
  
  path_parameters: {
    id: string;
  };
  
  request_body: {
    name?: string;
    configuration?: Record<string, any>;
    metadata?: Record<string, any>;
    status?: "active" | "inactive" | "maintenance";
  };
  
  response: {
    success: boolean;
    data: Agent;
    message: string;
    changes: string[]; // List of changed fields
  };
}

// DELETE /v1/agents/{id} - Delete agent
interface DeleteAgentEndpoint {
  method: "DELETE";
  path: "/v1/agents/{id}";
  
  path_parameters: {
    id: string;
  };
  
  query_parameters: {
    force?: boolean; // Force delete even if agent is active
  };
  
  response: {
    success: boolean;
    message: string;
  };
}
```

#### Workflow Management Endpoints
```typescript
// POST /v1/workflows - Create workflow
interface CreateWorkflowEndpoint {
  method: "POST";
  path: "/v1/workflows";
  
  request_body: {
    name: string;
    description?: string;
    steps: WorkflowStep[];
    triggers?: WorkflowTrigger[];
    configuration?: Record<string, any>;
    schedule?: CronExpression;
  };
  
  response: {
    success: boolean;
    data: Workflow;
    validation_results: ValidationResult[];
  };
}

// POST /v1/workflows/{id}/execute - Execute workflow
interface ExecuteWorkflowEndpoint {
  method: "POST";
  path: "/v1/workflows/{id}/execute";
  
  path_parameters: {
    id: string;
  };
  
  request_body: {
    input_data?: Record<string, any>;
    execution_context?: Record<string, any>;
    priority?: "low" | "normal" | "high" | "urgent";
    timeout?: number; // Execution timeout in seconds
  };
  
  response: {
    success: boolean;
    data: {
      execution_id: string;
      status: "queued" | "running" | "completed" | "failed";
      estimated_duration?: number;
      queue_position?: number;
    };
    message: string;
  };
}

// GET /v1/workflows/{id}/executions - Get workflow executions
interface GetWorkflowExecutionsEndpoint {
  method: "GET";
  path: "/v1/workflows/{id}/executions";
  
  path_parameters: {
    id: string;
  };
  
  query_parameters: {
    status?: "queued" | "running" | "completed" | "failed";
    start_date?: string; // ISO 8601 date
    end_date?: string;
    page?: number;
    limit?: number;
  };
  
  response: {
    success: boolean;
    data: WorkflowExecution[];
    pagination: PaginationInfo;
    summary: {
      total_executions: number;
      success_rate: number;
      average_duration: number;
    };
  };
}
```

#### Real-time Communication Endpoints
```typescript
// WebSocket connection for real-time updates
interface WebSocketAPI {
  endpoint: "wss://api.dafnckmachine.com/v1/ws";
  
  authentication: {
    method: "query_parameter";
    parameter: "token";
    example: "wss://api.dafnckmachine.com/v1/ws?token=<jwt_token>";
  };
  
  message_format: {
    type: string;
    channel: string;
    data: any;
    timestamp: string;
  };
  
  channels: {
    "agent.status": "Agent status updates";
    "workflow.execution": "Workflow execution updates";
    "system.alerts": "System alerts and notifications";
    "user.notifications": "User-specific notifications";
  };
  
  client_messages: {
    subscribe: {
      type: "subscribe";
      channel: string;
      filters?: Record<string, any>;
    };
    unsubscribe: {
      type: "unsubscribe";
      channel: string;
    };
    ping: {
      type: "ping";
    };
  };
  
  server_messages: {
    update: {
      type: "update";
      channel: string;
      data: any;
      timestamp: string;
    };
    error: {
      type: "error";
      message: string;
      code: string;
    };
    pong: {
      type: "pong";
    };
  };
}
```

### Response Format Standards

#### Standard Response Structure
```typescript
// Success response format
interface SuccessResponse<T> {
  success: true;
  data: T;
  message?: string;
  meta?: {
    timestamp: string;
    request_id: string;
    response_time: number;
    api_version: string;
  };
}

// Error response format
interface ErrorResponse {
  success: false;
  error: {
    code: string;
    message: string;
    details?: any;
    field_errors?: FieldError[];
  };
  meta: {
    timestamp: string;
    request_id: string;
    api_version: string;
  };
}

// Field validation error
interface FieldError {
  field: string;
  message: string;
  code: string;
  value?: any;
}

// Pagination response
interface PaginatedResponse<T> {
  success: true;
  data: T[];
  pagination: {
    page: number;
    limit: number;
    total: number;
    total_pages: number;
    has_next: boolean;
    has_prev: boolean;
    next_page?: number;
    prev_page?: number;
  };
  links?: {
    first: string;
    last: string;
    next?: string;
    prev?: string;
  };
}
```

#### HTTP Status Codes
```typescript
// Standard HTTP status codes usage
interface HTTPStatusCodes {
  // Success responses
  200: "OK - Request successful";
  201: "Created - Resource created successfully";
  202: "Accepted - Request accepted for processing";
  204: "No Content - Request successful, no content to return";
  
  // Client error responses
  400: "Bad Request - Invalid request format or parameters";
  401: "Unauthorized - Authentication required";
  403: "Forbidden - Insufficient permissions";
  404: "Not Found - Resource not found";
  409: "Conflict - Resource conflict (e.g., duplicate)";
  422: "Unprocessable Entity - Validation errors";
  429: "Too Many Requests - Rate limit exceeded";
  
  // Server error responses
  500: "Internal Server Error - Unexpected server error";
  502: "Bad Gateway - Upstream service error";
  503: "Service Unavailable - Service temporarily unavailable";
  504: "Gateway Timeout - Upstream service timeout";
}

// Error code mapping
interface ErrorCodes {
  // Authentication errors
  AUTH_TOKEN_MISSING: "Authentication token is required";
  AUTH_TOKEN_INVALID: "Invalid or expired authentication token";
  AUTH_TOKEN_EXPIRED: "Authentication token has expired";
  
  // Authorization errors
  INSUFFICIENT_PERMISSIONS: "Insufficient permissions for this operation";
  RESOURCE_ACCESS_DENIED: "Access denied to the requested resource";
  
  // Validation errors
  VALIDATION_FAILED: "Request validation failed";
  REQUIRED_FIELD_MISSING: "Required field is missing";
  INVALID_FIELD_FORMAT: "Field format is invalid";
  
  // Resource errors
  RESOURCE_NOT_FOUND: "Requested resource not found";
  RESOURCE_ALREADY_EXISTS: "Resource already exists";
  RESOURCE_CONFLICT: "Resource conflict detected";
  
  // Rate limiting errors
  RATE_LIMIT_EXCEEDED: "API rate limit exceeded";
  QUOTA_EXCEEDED: "API quota exceeded";
  
  // System errors
  INTERNAL_ERROR: "Internal server error occurred";
  SERVICE_UNAVAILABLE: "Service temporarily unavailable";
  UPSTREAM_ERROR: "Upstream service error";
}
```

### Rate Limiting & Throttling

#### Rate Limiting Strategy
```typescript
interface RateLimitingConfig {
  // Global rate limits
  global: {
    requests_per_minute: 10000;
    burst_capacity: 1000;
    window_size: "1m";
  };
  
  // Per-user rate limits
  per_user: {
    authenticated: {
      requests_per_minute: 1000;
      burst_capacity: 100;
      daily_quota: 100000;
    };
    anonymous: {
      requests_per_minute: 100;
      burst_capacity: 10;
      daily_quota: 1000;
    };
  };
  
  // Per-endpoint rate limits
  per_endpoint: {
    "/v1/agents/execute": {
      requests_per_minute: 50;
      burst_capacity: 10;
      cost_multiplier: 2; // Counts as 2 requests
    };
    "/v1/workflows/execute": {
      requests_per_minute: 30;
      burst_capacity: 5;
      cost_multiplier: 3;
    };
    "/v1/data/export": {
      requests_per_minute: 10;
      burst_capacity: 2;
      cost_multiplier: 5;
    };
  };
  
  // Rate limit headers
  response_headers: {
    "X-RateLimit-Limit": "Request limit per window";
    "X-RateLimit-Remaining": "Remaining requests in current window";
    "X-RateLimit-Reset": "Unix timestamp when limit resets";
    "X-RateLimit-Window": "Rate limit window size";
    "Retry-After": "Seconds to wait before retrying (when rate limited)";
  };
}

// Rate limiting response
interface RateLimitResponse {
  success: false;
  error: {
    code: "RATE_LIMIT_EXCEEDED";
    message: "API rate limit exceeded";
    details: {
      limit: number;
      window: string;
      reset_time: string;
      retry_after: number;
    };
  };
}
```

### API Documentation Standards

#### OpenAPI Specification
```yaml
# OpenAPI 3.0 specification structure
openapi: 3.0.3
info:
  title: DafnckMachine API
  description: |
    Comprehensive API for the DafnckMachine AI Agent Orchestration Platform.
    
    ## Authentication
    This API uses JWT Bearer tokens for authentication. Include the token in the Authorization header:
    ```
    Authorization: Bearer <your_jwt_token>
    ```
    
    ## Rate Limiting
    API requests are rate limited. Check the response headers for current limits:
    - X-RateLimit-Limit: Request limit per window
    - X-RateLimit-Remaining: Remaining requests
    - X-RateLimit-Reset: Reset timestamp
    
    ## Error Handling
    All errors follow a consistent format with appropriate HTTP status codes and detailed error messages.
  version: 1.0.0
  contact:
    name: DafnckMachine API Support
    url: https://docs.dafnckmachine.com
    email: api-support@dafnckmachine.com
  license:
    name: MIT
    url: https://opensource.org/licenses/MIT

servers:
  - url: https://api.dafnckmachine.com/v1
    description: Production server
  - url: https://api-staging.dafnckmachine.com/v1
    description: Staging server

security:
  - bearerAuth: []

components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
      description: JWT token obtained from authentication endpoint

  schemas:
    Agent:
      type: object
      required:
        - id
        - name
        - type
        - status
        - capabilities
      properties:
        id:
          type: string
          format: uuid
          description: Unique agent identifier
          example: "123e4567-e89b-12d3-a456-426614174000"
        name:
          type: string
          minLength: 1
          maxLength: 255
          description: Agent name
          example: "Data Processing Agent"
        type:
          type: string
          enum: [orchestrator, worker, monitor]
          description: Agent type
        status:
          type: string
          enum: [active, inactive, error, maintenance]
          description: Current agent status
        capabilities:
          type: array
          items:
            type: string
          minItems: 1
          description: Agent capabilities
          example: ["data-processing", "file-parsing"]
        configuration:
          type: object
          description: Agent configuration settings
        metadata:
          type: object
          description: Additional agent metadata
        created_at:
          type: string
          format: date-time
          description: Creation timestamp
        updated_at:
          type: string
          format: date-time
          description: Last update timestamp

    Error:
      type: object
      required:
        - success
        - error
      properties:
        success:
          type: boolean
          enum: [false]
        error:
          type: object
          required:
            - code
            - message
          properties:
            code:
              type: string
              description: Error code
            message:
              type: string
              description: Human-readable error message
            details:
              type: object
              description: Additional error details
            field_errors:
              type: array
              items:
                $ref: '#/components/schemas/FieldError'
        meta:
          $ref: '#/components/schemas/ResponseMeta'

    FieldError:
      type: object
      required:
        - field
        - message
        - code
      properties:
        field:
          type: string
          description: Field name that caused the error
        message:
          type: string
          description: Field-specific error message
        code:
          type: string
          description: Field error code
        value:
          description: Invalid value that caused the error

    ResponseMeta:
      type: object
      properties:
        timestamp:
          type: string
          format: date-time
          description: Response timestamp
        request_id:
          type: string
          description: Unique request identifier
        response_time:
          type: number
          description: Response time in milliseconds
        api_version:
          type: string
          description: API version used
```

### Integration Guidelines

#### SDK Development Standards
```typescript
// TypeScript SDK structure
interface DafnckMachineSDK {
  // Configuration
  configure(options: SDKOptions): void;
  
  // Agent management
  agents: {
    list(params?: ListAgentsParams): Promise<PaginatedResponse<Agent>>;
    get(id: string): Promise<Agent>;
    create(data: CreateAgentData): Promise<Agent>;
    update(id: string, data: UpdateAgentData): Promise<Agent>;
    delete(id: string): Promise<void>;
    execute(id: string, params: ExecuteParams): Promise<ExecutionResult>;
  };
  
  // Workflow management
  workflows: {
    list(params?: ListWorkflowsParams): Promise<PaginatedResponse<Workflow>>;
    get(id: string): Promise<Workflow>;
    create(data: CreateWorkflowData): Promise<Workflow>;
    execute(id: string, params: ExecuteWorkflowParams): Promise<ExecutionResult>;
    getExecutions(id: string, params?: GetExecutionsParams): Promise<PaginatedResponse<WorkflowExecution>>;
  };
  
  // Real-time communication
  realtime: {
    connect(): Promise<WebSocketConnection>;
    subscribe(channel: string, callback: (data: any) => void): void;
    unsubscribe(channel: string): void;
    disconnect(): void;
  };
  
  // Utilities
  utils: {
    validateToken(token: string): boolean;
    refreshToken(): Promise<string>;
    handleError(error: APIError): void;
  };
}

// SDK configuration options
interface SDKOptions {
  apiKey?: string;
  baseURL?: string;
  timeout?: number;
  retryAttempts?: number;
  retryDelay?: number;
  debug?: boolean;
  userAgent?: string;
}
```

#### Webhook Integration
```typescript
// Webhook configuration
interface WebhookConfig {
  endpoints: {
    agent_status_changed: {
      url: string;
      secret: string;
      events: ["agent.created", "agent.updated", "agent.deleted", "agent.status_changed"];
    };
    workflow_completed: {
      url: string;
      secret: string;
      events: ["workflow.started", "workflow.completed", "workflow.failed"];
    };
  };
  
  security: {
    signature_header: "X-DafnckMachine-Signature";
    signature_algorithm: "HMAC-SHA256";
    timestamp_header: "X-DafnckMachine-Timestamp";
    timestamp_tolerance: 300; // 5 minutes
  };
  
  retry_policy: {
    max_attempts: 3;
    backoff_strategy: "exponential";
    initial_delay: 1000; // 1 second
    max_delay: 30000; // 30 seconds
  };
}

// Webhook payload structure
interface WebhookPayload {
  event: string;
  timestamp: string;
  data: any;
  signature: string;
  delivery_id: string;
}
```

---

**Document Status**: Active  
**Last Updated**: 2025-01-27  
**Next Review**: 2025-02-27  
**Owner**: @system-architect-agent 