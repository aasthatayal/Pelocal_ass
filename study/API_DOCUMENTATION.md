# API Documentation

## Overview

The To-Do List API provides a complete set of RESTful endpoints for managing tasks. All endpoints accept and return JSON data.

**Base URL**: `http://localhost:8000`

## Authentication

Currently, the API does not require authentication. For production use, implement JWT or OAuth2.

## Response Format

All responses follow a consistent format:

**Success Response:**
```json
{
  "id": 1,
  "title": "Task title",
  "description": "Task description",
  "due_date": "2025-12-31",
  "status": "pending",
  "created_at": "2025-11-20T10:00:00",
  "updated_at": "2025-11-20T10:00:00"
}
```

**Error Response:**
```json
{
  "detail": "Error message"
}
```

## Status Codes

| Code | Description |
|------|-------------|
| 200 | OK - Request succeeded |
| 201 | Created - Resource created successfully |
| 204 | No Content - Request succeeded, no content to return |
| 400 | Bad Request - Invalid input |
| 404 | Not Found - Resource not found |
| 422 | Unprocessable Entity - Validation error |
| 500 | Internal Server Error - Server error |

## Endpoints

### 1. Health Check

Check if the API and database are running.

**Endpoint:** `GET /health`

**Response:**
```json
{
  "status": "healthy",
  "database": "connected"
}
```

---

### 2. Create Task

Create a new task.

**Endpoint:** `POST /api/tasks`

**Request Body:**
```json
{
  "title": "Complete FastAPI tutorial",
  "description": "Learn FastAPI basics and advanced features",
  "due_date": "2025-11-25",
  "status": "pending"
}
```

**Field Descriptions:**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| title | string | Yes | Task title | 1-255 characters |
| description | string | No | Task description | Any length |
| due_date | date | No | Due date | ISO 8601 format (YYYY-MM-DD) |
| status | string | No | Task status | pending, in_progress, completed (default: pending) |

**Success Response (201):**
```json
{
  "id": 1,
  "title": "Complete FastAPI tutorial",
  "description": "Learn FastAPI basics and advanced features",
  "due_date": "2025-11-25",
  "status": "pending",
  "created_at": "2025-11-20T10:30:00",
  "updated_at": "2025-11-20T10:30:00"
}
```

**Error Responses:**

- **400 Bad Request**: Invalid status value
- **422 Validation Error**: Missing or invalid fields
- **500 Internal Server Error**: Database error

**Example:**
```bash
curl -X POST http://localhost:8000/api/tasks \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Complete FastAPI tutorial",
    "description": "Learn FastAPI basics and advanced features",
    "due_date": "2025-11-25",
    "status": "pending"
  }'
```

---

### 3. Get All Tasks

Retrieve all tasks, optionally filtered by status.

**Endpoint:** `GET /api/tasks`

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| status | string | No | Filter by status (pending, in_progress, completed) |

**Success Response (200):**
```json
{
  "tasks": [
    {
      "id": 1,
      "title": "Complete FastAPI tutorial",
      "description": "Learn FastAPI basics and advanced features",
      "due_date": "2025-11-25",
      "status": "in_progress",
      "created_at": "2025-11-20T10:30:00",
      "updated_at": "2025-11-20T15:45:00"
    },
    {
      "id": 2,
      "title": "Setup PostgreSQL database",
      "description": "Install and configure PostgreSQL",
      "due_date": "2025-11-22",
      "status": "completed",
      "created_at": "2025-11-20T09:00:00",
      "updated_at": "2025-11-20T14:30:00"
    }
  ],
  "count": 2
}
```

**Error Responses:**

- **400 Bad Request**: Invalid status parameter
- **500 Internal Server Error**: Database error

**Examples:**

Get all tasks:
```bash
curl http://localhost:8000/api/tasks
```

Get only pending tasks:
```bash
curl http://localhost:8000/api/tasks?status=pending
```

Get only completed tasks:
```bash
curl http://localhost:8000/api/tasks?status=completed
```

---

### 4. Get Task by ID

Retrieve a specific task by its ID.

**Endpoint:** `GET /api/tasks/{task_id}`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| task_id | integer | Yes | Task ID |

**Success Response (200):**
```json
{
  "id": 1,
  "title": "Complete FastAPI tutorial",
  "description": "Learn FastAPI basics and advanced features",
  "due_date": "2025-11-25",
  "status": "in_progress",
  "created_at": "2025-11-20T10:30:00",
  "updated_at": "2025-11-20T15:45:00"
}
```

**Error Responses:**

- **404 Not Found**: Task with specified ID does not exist
- **500 Internal Server Error**: Database error

**Example:**
```bash
curl http://localhost:8000/api/tasks/1
```

---

### 5. Update Task

Update an existing task. All fields are optional (partial update supported).

**Endpoint:** `PUT /api/tasks/{task_id}`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| task_id | integer | Yes | Task ID |

**Request Body:**
```json
{
  "title": "Updated task title",
  "description": "Updated description",
  "due_date": "2025-12-31",
  "status": "completed"
}
```

**Field Descriptions:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| title | string | No | New task title |
| description | string | No | New task description |
| due_date | date | No | New due date |
| status | string | No | New status (pending, in_progress, completed) |

**Success Response (200):**
```json
{
  "id": 1,
  "title": "Updated task title",
  "description": "Updated description",
  "due_date": "2025-12-31",
  "status": "completed",
  "created_at": "2025-11-20T10:30:00",
  "updated_at": "2025-11-20T16:45:00"
}
```

**Error Responses:**

- **404 Not Found**: Task with specified ID does not exist
- **422 Validation Error**: Invalid field values
- **500 Internal Server Error**: Database error

**Examples:**

Update only status:
```bash
curl -X PUT http://localhost:8000/api/tasks/1 \
  -H "Content-Type: application/json" \
  -d '{"status": "completed"}'
```

Update multiple fields:
```bash
curl -X PUT http://localhost:8000/api/tasks/1 \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Updated title",
    "status": "in_progress",
    "due_date": "2025-12-31"
  }'
```

---

### 6. Delete Task

Delete a task by its ID.

**Endpoint:** `DELETE /api/tasks/{task_id}`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| task_id | integer | Yes | Task ID |

**Success Response (204):**
No content returned.

**Error Responses:**

- **404 Not Found**: Task with specified ID does not exist
- **500 Internal Server Error**: Database error

**Example:**
```bash
curl -X DELETE http://localhost:8000/api/tasks/1
```

---

## Task Status Values

| Status | Description |
|--------|-------------|
| pending | Task has not been started yet |
| in_progress | Task is currently being worked on |
| completed | Task has been finished |

---

## Data Validation

### Title Validation
- **Required**: Yes
- **Min Length**: 1 character (after trimming)
- **Max Length**: 255 characters
- **Cannot be empty or only whitespace**

### Description Validation
- **Required**: No
- **Type**: Text
- **Max Length**: Unlimited

### Due Date Validation
- **Required**: No
- **Format**: ISO 8601 date (YYYY-MM-DD)
- **Example**: "2025-12-31"

### Status Validation
- **Required**: No (defaults to "pending")
- **Allowed Values**: "pending", "in_progress", "completed"
- **Case Sensitive**: Must be lowercase with underscore

---

## Error Handling

### Validation Errors (422)

When invalid data is provided:

```json
{
  "detail": [
    {
      "loc": ["body", "title"],
      "msg": "field required",
      "type": "value_error.missing"
    }
  ]
}
```

### Not Found Errors (404)

When a resource doesn't exist:

```json
{
  "detail": "Task with ID 999 not found"
}
```

### Bad Request Errors (400)

When invalid query parameters are provided:

```json
{
  "detail": "Invalid status. Must be one of: pending, in_progress, completed"
}
```

### Internal Server Errors (500)

When an unexpected error occurs:

```json
{
  "detail": "Internal server error: <error message>"
}
```

---

## Rate Limiting

Currently, no rate limiting is implemented. For production, consider:

- **Rate limit**: 100 requests per minute per IP
- **Burst**: 20 requests
- **Response header**: `X-RateLimit-Remaining`

---

## CORS

CORS is not configured. For production with frontend on different domain:

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://yourdomain.com"],
    allow_methods=["*"],
    allow_headers=["*"],
)
```

---

## Pagination

Not currently implemented. For large datasets, consider adding:

```
GET /api/tasks?page=1&per_page=10
```

Response:
```json
{
  "tasks": [...],
  "count": 100,
  "page": 1,
  "per_page": 10,
  "total_pages": 10
}
```

---

## Interactive Documentation

Visit these URLs when the server is running:

- **Swagger UI**: http://localhost:8000/docs
  - Interactive API testing
  - Try-it-out functionality
  - Request/response examples

- **ReDoc**: http://localhost:8000/redoc
  - Clean, readable documentation
  - Organized by tags
  - Search functionality

---

## Testing with Postman

Import the following collection:

1. Create a new collection named "To-Do List API"
2. Add requests for each endpoint
3. Set base URL variable: `{{baseUrl}}` = `http://localhost:8000`
4. Create test scenarios for success and error cases

---

## SDKs and Client Libraries

Currently, no official SDKs are provided. Generate clients using OpenAPI spec:

```bash
# Download OpenAPI spec
curl http://localhost:8000/openapi.json > openapi.json

# Generate Python client
openapi-generator generate -i openapi.json -g python -o client/

# Generate JavaScript client
openapi-generator generate -i openapi.json -g javascript -o client/
```

---

## Changelog

### Version 1.0.0 (2025-11-20)
- Initial API release
- CRUD operations for tasks
- Status filtering
- Health check endpoint
- Comprehensive error handling
- Interactive documentation

---

## Support

For issues or questions:
1. Check the interactive documentation at `/docs`
2. Review this documentation
3. Check the README.md file
4. Review the study guide in `study/study.md`

---

**API Documentation Version**: 1.0.0
**Last Updated**: November 2025
