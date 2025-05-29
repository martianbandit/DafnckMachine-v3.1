# API Documentation Complete - DafnckMachine v3.1

## Overview
Comprehensive API documentation for DafnckMachine v3.1 backend services, providing detailed endpoint specifications, authentication requirements, request/response schemas, and usage examples.

## API Base Information
- **Base URL**: `https://api.dafnckmachine.com/v3.1`
- **Protocol**: HTTPS
- **Authentication**: JWT Bearer Token
- **Content Type**: `application/json`
- **API Version**: 3.1.0

## Authentication

### JWT Token Authentication
All API endpoints require authentication via JWT Bearer token in the Authorization header.

```http
Authorization: Bearer <jwt_token>
```

### Authentication Endpoints

#### POST /auth/login
User authentication and token generation.

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "securePassword123"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "user": {
      "id": "user_123",
      "email": "user@example.com",
      "role": "user"
    },
    "expiresIn": 3600
  }
}
```

#### POST /auth/register
User registration and account creation.

**Request Body:**
```json
{
  "email": "newuser@example.com",
  "password": "securePassword123",
  "firstName": "John",
  "lastName": "Doe"
}
```

#### POST /auth/refresh
Token refresh for extended sessions.

#### POST /auth/logout
User logout and token invalidation.

## Core API Endpoints

### User Management

#### GET /users/profile
Retrieve current user profile information.

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "user_123",
    "email": "user@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "user",
    "createdAt": "2025-01-27T10:00:00Z",
    "lastLogin": "2025-01-27T15:30:00Z"
  }
}
```

#### PUT /users/profile
Update user profile information.

#### DELETE /users/account
Delete user account and associated data.

### Project Management

#### GET /projects
Retrieve user's projects with pagination.

**Query Parameters:**
- `page`: Page number (default: 1)
- `limit`: Items per page (default: 20)
- `status`: Filter by project status
- `search`: Search projects by name

**Response:**
```json
{
  "success": true,
  "data": {
    "projects": [
      {
        "id": "proj_123",
        "name": "My Project",
        "description": "Project description",
        "status": "active",
        "createdAt": "2025-01-27T10:00:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 45,
      "totalPages": 3
    }
  }
}
```

#### POST /projects
Create new project.

#### GET /projects/{id}
Retrieve specific project details.

#### PUT /projects/{id}
Update project information.

#### DELETE /projects/{id}
Delete project and associated data.

### Task Management

#### GET /projects/{projectId}/tasks
Retrieve project tasks.

#### POST /projects/{projectId}/tasks
Create new task in project.

#### GET /tasks/{id}
Retrieve specific task details.

#### PUT /tasks/{id}
Update task information.

#### DELETE /tasks/{id}
Delete task.

### File Management

#### POST /files/upload
Upload files to project.

#### GET /files/{id}
Retrieve file information.

#### DELETE /files/{id}
Delete file.

## Error Handling

### Error Response Format
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": {
      "field": "email",
      "issue": "Invalid email format"
    }
  }
}
```

### HTTP Status Codes
- `200`: Success
- `201`: Created
- `400`: Bad Request
- `401`: Unauthorized
- `403`: Forbidden
- `404`: Not Found
- `422`: Validation Error
- `429`: Rate Limited
- `500`: Internal Server Error

### Common Error Codes
- `INVALID_TOKEN`: JWT token is invalid or expired
- `VALIDATION_ERROR`: Request data validation failed
- `RESOURCE_NOT_FOUND`: Requested resource doesn't exist
- `PERMISSION_DENIED`: Insufficient permissions
- `RATE_LIMIT_EXCEEDED`: Too many requests

## Rate Limiting
- **Rate Limit**: 1000 requests per hour per user
- **Headers**: 
  - `X-RateLimit-Limit`: Request limit
  - `X-RateLimit-Remaining`: Remaining requests
  - `X-RateLimit-Reset`: Reset timestamp

## Webhooks

### Webhook Events
- `project.created`
- `project.updated`
- `project.deleted`
- `task.created`
- `task.updated`
- `task.completed`

### Webhook Payload
```json
{
  "event": "project.created",
  "data": {
    "id": "proj_123",
    "name": "New Project",
    "userId": "user_123"
  },
  "timestamp": "2025-01-27T15:30:00Z"
}
```

## SDK Examples

### JavaScript/Node.js
```javascript
const DafnckAPI = require('@dafnckmachine/api-client');

const client = new DafnckAPI({
  apiKey: 'your-api-key',
  baseURL: 'https://api.dafnckmachine.com/v3.1'
});

// Get user profile
const profile = await client.users.getProfile();

// Create project
const project = await client.projects.create({
  name: 'My New Project',
  description: 'Project description'
});
```

### Python
```python
from dafnckmachine import DafnckAPI

client = DafnckAPI(
    api_key='your-api-key',
    base_url='https://api.dafnckmachine.com/v3.1'
)

# Get user profile
profile = client.users.get_profile()

# Create project
project = client.projects.create(
    name='My New Project',
    description='Project description'
)
```

## Testing

### Postman Collection
Import the Postman collection for interactive API testing:
- Collection URL: `https://api.dafnckmachine.com/postman/collection.json`

### API Testing Environment
- **Staging**: `https://staging-api.dafnckmachine.com/v3.1`
- **Development**: `https://dev-api.dafnckmachine.com/v3.1`

## Changelog

### Version 3.1.0
- Added project management endpoints
- Implemented task management system
- Enhanced authentication with refresh tokens
- Added file upload capabilities
- Improved error handling and validation

### Version 3.0.0
- Initial API release
- Basic user authentication
- Core project functionality

## Support
- **Documentation**: https://docs.dafnckmachine.com
- **Support Email**: api-support@dafnckmachine.com
- **Status Page**: https://status.dafnckmachine.com 