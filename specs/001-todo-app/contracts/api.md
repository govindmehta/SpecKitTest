# API Contracts: Todo Application

**Version**: 1.0  
**Date**: 2026-01-05  
**Base URL**: `{API_HOST}/api` (e.g., `https://api.example.com/api`)

## Overview

This document defines the REST API contracts for the Todo application. All endpoints use JSON for request/response bodies and follow REST conventions.

---

## Authentication Endpoints

### Register User

**Endpoint**: `POST /auth/register`

**Description**: Create a new user account.

**Request**:
```json
{
  "email": "user@example.com",
  "password": "securePassword123"
}
```

**Success Response** (201 Created):
```json
{
  "user": {
    "id": "user_abc123",
    "email": "user@example.com",
    "createdAt": "2026-01-05T10:00:00.000Z"
  },
  "accessToken": "eyJhbGciOiJIUzI1NiIs...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIs..."
}
```

**Error Responses**:
- `400 Bad Request`: Invalid input
  ```json
  {
    "error": {
      "code": "VALIDATION_ERROR",
      "message": "Invalid input data",
      "details": [
        { "field": "email", "issue": "Invalid email format" },
        { "field": "password", "issue": "Must be 8-64 characters" }
      ]
    }
  }
  ```
- `409 Conflict`: Email already registered
  ```json
  {
    "error": {
      "code": "EMAIL_EXISTS",
      "message": "Email already registered"
    }
  }
  ```

---

### Login

**Endpoint**: `POST /auth/login`

**Description**: Authenticate user and receive tokens.

**Request**:
```json
{
  "email": "user@example.com",
  "password": "securePassword123"
}
```

**Success Response** (200 OK):
```json
{
  "user": {
    "id": "user_abc123",
    "email": "user@example.com"
  },
  "accessToken": "eyJhbGciOiJIUzI1NiIs...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIs..."
}
```

**Error Responses**:
- `401 Unauthorized`: Invalid credentials
  ```json
  {
    "error": {
      "code": "INVALID_CREDENTIALS",
      "message": "Invalid email or password"
    }
  }
  ```

---

### Refresh Token

**Endpoint**: `POST /auth/refresh`

**Description**: Obtain new access token using refresh token.

**Request Headers**:
```
Cookie: refreshToken=eyJhbGciOiJIUzI1NiIs...
```

**Success Response** (200 OK):
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIs..."
}
```

**Error Responses**:
- `401 Unauthorized`: Invalid or expired refresh token
  ```json
  {
    "error": {
      "code": "INVALID_TOKEN",
      "message": "Invalid or expired refresh token"
    }
  }
  ```

---

### Logout

**Endpoint**: `POST /auth/logout`

**Description**: Invalidate refresh token.

**Request Headers**:
```
Authorization: Bearer {accessToken}
```

**Success Response** (204 No Content): Empty body

---

## Task Endpoints

All task endpoints require authentication via `Authorization: Bearer {accessToken}` header.

### List Tasks

**Endpoint**: `GET /tasks`

**Description**: Retrieve all tasks for authenticated user.

**Query Parameters**:
- `completed` (optional): Filter by completion status (`true`, `false`, or omit for all)
- `limit` (optional): Number of tasks per page (default: 50, max: 100)
- `cursor` (optional): Pagination cursor (createdAt timestamp)

**Example Request**:
```
GET /api/tasks?completed=false&limit=20
```

**Success Response** (200 OK):
```json
{
  "tasks": [
    {
      "id": "task_xyz789",
      "description": "Buy groceries",
      "isComplete": false,
      "order": 0,
      "createdAt": "2026-01-05T10:00:00.000Z",
      "completedAt": null,
      "updatedAt": "2026-01-05T10:00:00.000Z"
    },
    {
      "id": "task_abc456",
      "description": "Walk the dog",
      "isComplete": false,
      "order": 1,
      "createdAt": "2026-01-04T15:30:00.000Z",
      "completedAt": null,
      "updatedAt": "2026-01-04T15:30:00.000Z"
    }
  ],
  "pagination": {
    "hasMore": false,
    "nextCursor": null
  }
}
```

**Error Responses**:
- `401 Unauthorized`: Missing or invalid token

---

### Create Task

**Endpoint**: `POST /tasks`

**Description**: Create a new task for authenticated user.

**Request**:
```json
{
  "description": "Buy groceries"
}
```

**Success Response** (201 Created):
```json
{
  "task": {
    "id": "task_xyz789",
    "description": "Buy groceries",
    "isComplete": false,
    "order": 0,
    "createdAt": "2026-01-05T10:00:00.000Z",
    "completedAt": null,
    "updatedAt": "2026-01-05T10:00:00.000Z"
  }
}
```

**Error Responses**:
- `400 Bad Request`: Invalid input
  ```json
  {
    "error": {
      "code": "VALIDATION_ERROR",
      "message": "Task description is required",
      "details": [
        { "field": "description", "issue": "Must not be empty" }
      ]
    }
  }
  ```
- `401 Unauthorized`: Missing or invalid token

---

### Get Single Task

**Endpoint**: `GET /tasks/:id`

**Description**: Retrieve a specific task (unused in MVP but included for completeness).

**Success Response** (200 OK):
```json
{
  "task": {
    "id": "task_xyz789",
    "description": "Buy groceries",
    "isComplete": false,
    "order": 0,
    "createdAt": "2026-01-05T10:00:00.000Z",
    "completedAt": null,
    "updatedAt": "2026-01-05T10:00:00.000Z"
  }
}
```

**Error Responses**:
- `403 Forbidden`: Task belongs to another user
- `404 Not Found`: Task does not exist

---

### Update Task

**Endpoint**: `PUT /tasks/:id`

**Description**: Update task description, completion status, or order.

**Request**:
```json
{
  "description": "Buy groceries and milk",
  "isComplete": true,
  "order": 5,
  "updatedAt": "2026-01-05T10:00:00.000Z"
}
```

**Notes**:
- All fields are optional; only provided fields are updated
- `updatedAt` is required for conflict detection (optimistic locking)

**Success Response** (200 OK):
```json
{
  "task": {
    "id": "task_xyz789",
    "description": "Buy groceries and milk",
    "isComplete": true,
    "order": 5,
    "createdAt": "2026-01-05T10:00:00.000Z",
    "completedAt": "2026-01-05T11:00:00.000Z",
    "updatedAt": "2026-01-05T11:00:00.000Z"
  }
}
```

**Error Responses**:
- `400 Bad Request`: Invalid input
  ```json
  {
    "error": {
      "code": "VALIDATION_ERROR",
      "message": "Description must be 1-500 characters"
    }
  }
  ```
- `403 Forbidden`: Task belongs to another user
- `404 Not Found`: Task does not exist
- `409 Conflict`: Task was modified in another session
  ```json
  {
    "error": {
      "code": "CONFLICT",
      "message": "Task was modified in another session. Please refresh and try again.",
      "latestVersion": {
        "description": "Buy groceries and bread",
        "updatedAt": "2026-01-05T10:30:00.000Z"
      }
    }
  }
  ```

---

### Delete Task

**Endpoint**: `DELETE /tasks/:id`

**Description**: Permanently delete a task.

**Success Response** (204 No Content): Empty body

**Error Responses**:
- `403 Forbidden`: Task belongs to another user
- `404 Not Found`: Task does not exist

---

## Global Error Responses

All endpoints may return these error responses:

### 500 Internal Server Error
```json
{
  "error": {
    "code": "INTERNAL_ERROR",
    "message": "An unexpected error occurred. Please try again."
  }
}
```

### 503 Service Unavailable
```json
{
  "error": {
    "code": "SERVICE_UNAVAILABLE",
    "message": "Service temporarily unavailable. Please try again later."
  }
}
```

---

## Response Headers

All responses include:
```
Content-Type: application/json; charset=utf-8
X-Request-Id: {unique-request-id}
```

Authenticated endpoints also include:
```
X-Rate-Limit-Remaining: 100
X-Rate-Limit-Reset: 1609459200
```

---

## Rate Limiting

**Limits**:
- Anonymous (register/login): 5 requests per minute per IP
- Authenticated: 100 requests per minute per user

**Exceeded Response** (429 Too Many Requests):
```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests. Please try again later.",
    "retryAfter": 60
  }
}
```

---

## CORS Configuration

**Allowed Origins**: Frontend domain (configured via environment variable)  
**Allowed Methods**: GET, POST, PUT, DELETE, OPTIONS  
**Allowed Headers**: Authorization, Content-Type  
**Credentials**: true (for cookies)

---

## Versioning

**Current Version**: 1.0

**Breaking Changes Policy**:
- Version increment required for incompatible changes
- New version would use `/api/v2` prefix
- v1 maintained for 6 months after v2 release

---

## Testing Contract

See [api-tests.http](api-tests.http) for example requests and expected responses.
