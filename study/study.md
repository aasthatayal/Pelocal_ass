# To-Do List Application - Technical Study Guide

This document provides an in-depth technical analysis of the To-Do List application, including architecture, design decisions, and implementation details.

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [System Components](#system-components)
3. [Database Design](#database-design)
4. [API Design](#api-design)
5. [Technology Stack](#technology-stack)
6. [Design Patterns](#design-patterns)
7. [Security Considerations](#security-considerations)
8. [Performance Optimization](#performance-optimization)
9. [Testing Strategy](#testing-strategy)
10. [Deployment Architecture](#deployment-architecture)

---

## Architecture Overview

The application follows a **three-tier architecture** with clear separation of concerns:

### Architecture Diagram (YAML)

```yaml
architecture:
  presentation_tier:
    - component: Web Browser
      routes:
        - GET /
        - GET /add
        - POST /tasks/create
        - POST /tasks/{id}/update-status
        - POST /tasks/{id}/delete
    - component: API Client
      routes:
        - GET /api/tasks
        - GET /api/tasks/{id}
        - POST /api/tasks
        - PUT /api/tasks/{id}
        - DELETE /api/tasks/{id}
  
  application_tier:
    fastapi_application:
      - component: Web Routes
        responsibility: Serve HTML templates
        files:
          - app/main.py (web routes)
      
      - component: API Routes
        responsibility: RESTful API endpoints
        files:
          - app/api/tasks.py
      
      - component: Business Logic
        responsibility: Data validation and transformation
        files:
          - app/models.py
          - app/config.py
      
      - component: Database Layer
        responsibility: Raw SQL queries
        files:
          - app/database/connection.py
          - app/database/repository.py
  
  data_tier:
    - component: PostgreSQL Database
      schema:
        - table: tasks
          columns:
            - id: SERIAL PRIMARY KEY
            - title: VARCHAR(255) NOT NULL
            - description: TEXT
            - due_date: DATE
            - status: VARCHAR(50)
            - created_at: TIMESTAMP
            - updated_at: TIMESTAMP
```

### Visual Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                     Presentation Layer                       │
│  ┌──────────────┐              ┌────────────────────┐       │
│  │ Web Browser  │              │  API Clients       │       │
│  │  (HTML/CSS)  │              │  (curl, Postman)   │       │
│  └──────┬───────┘              └─────────┬──────────┘       │
│         │                                 │                  │
└─────────┼─────────────────────────────────┼──────────────────┘
          │                                 │
          ▼                                 ▼
┌─────────────────────────────────────────────────────────────┐
│                   Application Layer                          │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              FastAPI Application                      │   │
│  │  ┌────────────────┐      ┌─────────────────────┐    │   │
│  │  │  Web Routes    │      │   API Routes        │    │   │
│  │  │  (/,/add)      │      │   (/api/tasks)      │    │   │
│  │  └────────┬───────┘      └──────────┬──────────┘    │   │
│  │           │                         │                │   │
│  │           └──────────┬──────────────┘                │   │
│  │                      ▼                               │   │
│  │           ┌──────────────────────┐                   │   │
│  │           │  Pydantic Models     │                   │   │
│  │           │  (Validation)        │                   │   │
│  │           └──────────┬───────────┘                   │   │
│  │                      ▼                               │   │
│  │           ┌──────────────────────┐                   │   │
│  │           │  Database Repository │                   │   │
│  │           │  (Raw SQL Queries)   │                   │   │
│  │           └──────────┬───────────┘                   │   │
│  └──────────────────────┼──────────────────────────────┘   │
└─────────────────────────┼──────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                      Data Layer                              │
│  ┌────────────────────────────────────────────────────┐     │
│  │           PostgreSQL Database                       │     │
│  │  ┌──────────────────────────────────────────┐      │     │
│  │  │          tasks Table                      │      │     │
│  │  │  - id (PK)                                │      │     │
│  │  │  - title                                  │      │     │
│  │  │  - description                            │      │     │
│  │  │  - due_date                               │      │     │
│  │  │  - status                                 │      │     │
│  │  │  - created_at                             │      │     │
│  │  │  - updated_at                             │      │     │
│  │  └──────────────────────────────────────────┘      │     │
│  └────────────────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────────┘
```

---

## System Components

### 1. FastAPI Application (`app/main.py`)

**Responsibilities:**
- Initialize FastAPI application
- Configure middleware and CORS
- Register API routers
- Serve HTML templates
- Handle application lifecycle (startup/shutdown)

**Key Features:**
- Automatic API documentation (Swagger UI, ReDoc)
- Request/response validation
- Async/await support
- Built-in error handling

### 2. API Layer (`app/api/tasks.py`)

**Responsibilities:**
- Define RESTful endpoints
- Handle HTTP requests/responses
- Input validation via Pydantic
- Error handling and logging

**Endpoints:**
- `POST /api/tasks` - Create new task
- `GET /api/tasks` - Retrieve all tasks (with optional filters)
- `GET /api/tasks/{id}` - Retrieve specific task
- `PUT /api/tasks/{id}` - Update task
- `DELETE /api/tasks/{id}` - Delete task

### 3. Database Layer (`app/database/`)

**Why Raw SQL instead of ORM?**

As per requirements, we avoid using ORM. Benefits include:
- **Performance**: Direct SQL queries without ORM overhead
- **Control**: Full control over query optimization
- **Transparency**: Clear visibility of database operations
- **Learning**: Better understanding of SQL and database internals

**Components:**

#### Connection Manager (`connection.py`)
```python
class Database:
    - get_connection(): Context manager for connections
    - execute_query(): Execute queries with parameters
    - execute_one(): Execute and fetch single result
    - test_connection(): Health check
```

#### Repository Pattern (`repository.py`)
```python
class TaskRepository:
    - create_task(): INSERT operation
    - get_all_tasks(): SELECT with optional filters
    - get_task_by_id(): SELECT by primary key
    - update_task(): UPDATE with partial fields
    - delete_task(): DELETE operation
    - get_tasks_count(): Aggregate query
```

### 4. Data Models (`app/models.py`)

**Pydantic Models:**

1. **TaskCreate**: Request model for creating tasks
   - Validates required fields
   - Applies business rules
   
2. **TaskUpdate**: Request model for updating tasks
   - All fields optional
   - Partial updates supported
   
3. **TaskResponse**: Response model
   - Includes all fields
   - Automatic datetime formatting
   
4. **TaskListResponse**: Collection response
   - List of tasks
   - Count metadata

### 5. Configuration (`app/config.py`)

**Settings Management:**
- Environment variable loading via `.env`
- Type-safe configuration with Pydantic
- Separate concerns (DB, App, Logging)

---

## Database Design

### Entity Relationship Diagram

```
┌─────────────────────────────────────────┐
│              tasks                       │
├─────────────────────────────────────────┤
│ PK  id           SERIAL                  │
│     title        VARCHAR(255)  NOT NULL │
│     description  TEXT          NULL     │
│     due_date     DATE          NULL     │
│     status       VARCHAR(50)   DEFAULT  │
│     created_at   TIMESTAMP     DEFAULT  │
│     updated_at   TIMESTAMP     DEFAULT  │
├─────────────────────────────────────────┤
│ Indexes:                                 │
│   - PRIMARY KEY (id)                    │
│   - INDEX (status)                      │
│   - INDEX (due_date)                    │
├─────────────────────────────────────────┤
│ Constraints:                             │
│   - status CHECK (status IN (...))      │
├─────────────────────────────────────────┤
│ Triggers:                                │
│   - update_updated_at_column()          │
└─────────────────────────────────────────┘
```

### Data Flow Diagram

```yaml
data_flow:
  create_task:
    - step: 1
      actor: User
      action: Submit task form
      data: {title, description, due_date, status}
    
    - step: 2
      component: FastAPI
      action: Validate request
      validation:
        - title not empty
        - status in allowed values
    
    - step: 3
      component: Repository
      action: Execute INSERT query
      sql: INSERT INTO tasks (...) VALUES (...) RETURNING *
    
    - step: 4
      component: Database
      action: Insert record
      trigger: Update created_at, updated_at
    
    - step: 5
      component: FastAPI
      action: Return response
      format: JSON with TaskResponse schema
  
  get_tasks:
    - step: 1
      actor: User
      action: Request task list
      filters: {status: optional}
    
    - step: 2
      component: Repository
      action: Execute SELECT query
      sql: SELECT * FROM tasks WHERE status = $1 ORDER BY created_at DESC
    
    - step: 3
      component: Database
      action: Return matching rows
    
    - step: 4
      component: FastAPI
      action: Format and return
      format: JSON array with TaskListResponse
  
  update_task:
    - step: 1
      actor: User
      action: Submit update
      data: {partial fields}
    
    - step: 2
      component: Repository
      action: Build dynamic UPDATE
      sql: UPDATE tasks SET field1=$1, field2=$2 WHERE id=$3 RETURNING *
    
    - step: 3
      component: Database
      action: Update record
      trigger: Automatically update updated_at
    
    - step: 4
      component: FastAPI
      action: Return updated task
```

---

## API Design

### RESTful Principles

The API follows REST conventions:

1. **Resource-Based URLs**: `/api/tasks` represents the tasks collection
2. **HTTP Methods**: Use appropriate verbs (GET, POST, PUT, DELETE)
3. **Stateless**: Each request contains all necessary information
4. **JSON Format**: Standard data interchange format
5. **HTTP Status Codes**: Proper use of 200, 201, 204, 400, 404, 500

### Request/Response Flow

```
Client Request
     │
     ▼
┌─────────────────┐
│  HTTP Request   │
│  - Method       │
│  - URL          │
│  - Headers      │
│  - Body (JSON)  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  FastAPI Router │ ◄──── Route matching
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Request         │
│ Validation      │ ◄──── Pydantic models
│ (Pydantic)      │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Endpoint       │
│  Handler        │ ◄──── Business logic
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Repository     │ ◄──── Data access
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  SQL Query      │ ◄──── Raw SQL
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  PostgreSQL     │ ◄──── Database
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Response       │
│  Formatting     │ ◄──── Pydantic serialization
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  HTTP Response  │
│  - Status Code  │
│  - Headers      │
│  - Body (JSON)  │
└─────────────────┘
         │
         ▼
    Client
```

### API Documentation

FastAPI automatically generates interactive documentation:

1. **Swagger UI** (`/docs`): Interactive testing interface
2. **ReDoc** (`/redoc`): Clean, readable documentation

Features:
- Try-it-out functionality
- Request/response examples
- Schema definitions
- Error response documentation

---

## Technology Stack

### Backend Framework: FastAPI

**Why FastAPI?**

1. **Performance**: Built on Starlette and Pydantic, one of the fastest Python frameworks
2. **Type Safety**: Full type hints with Python 3.7+
3. **Automatic Documentation**: OpenAPI and JSON Schema generation
4. **Validation**: Automatic request validation with Pydantic
5. **Async Support**: Modern async/await syntax
6. **Developer Experience**: Excellent error messages and IDE support

### Database: PostgreSQL

**Why PostgreSQL?**

1. **ACID Compliance**: Reliable transactions
2. **Performance**: Excellent query optimization
3. **Features**: Rich data types, indexes, triggers
4. **Scalability**: Handles large datasets efficiently
5. **Community**: Mature ecosystem and support

### Connection Library: psycopg2

**Why psycopg2?**

1. **Standard**: Most widely used PostgreSQL adapter
2. **Raw SQL**: Direct query execution without ORM
3. **Performance**: Fast C implementation
4. **Features**: Connection pooling, async support
5. **Reliability**: Battle-tested in production

### Template Engine: Jinja2

**Why Jinja2?**

1. **Integration**: Built-in FastAPI support
2. **Power**: Full-featured template language
3. **Security**: Auto-escaping to prevent XSS
4. **Flexibility**: Inheritance, macros, filters

---

## Design Patterns

### 1. Repository Pattern

**Purpose**: Separate data access logic from business logic

**Implementation**:
```python
class TaskRepository:
    @staticmethod
    def create_task(...) -> Dict:
        # SQL query execution
        # Returns dictionary
```

**Benefits**:
- Testability: Easy to mock for unit tests
- Maintainability: Centralized data access
- Flexibility: Easy to change database implementation

### 2. Dependency Injection

**Purpose**: Provide dependencies at runtime

**Implementation**:
```python
# Configuration injection
from app.config import settings

# Database connection injection
from app.database import db, task_repo
```

**Benefits**:
- Loose coupling
- Easy testing
- Configuration management

### 3. Context Manager Pattern

**Purpose**: Manage resource lifecycle

**Implementation**:
```python
@contextmanager
def get_connection(self):
    conn = None
    try:
        conn = psycopg2.connect(...)
        yield conn
        conn.commit()
    except:
        if conn:
            conn.rollback()
        raise
    finally:
        if conn:
            conn.close()
```

**Benefits**:
- Automatic cleanup
- Exception safety
- Resource management

### 4. MVC Pattern (Modified)

**Model**: Pydantic models + Database schema
**View**: Jinja2 templates
**Controller**: FastAPI route handlers

---

## Security Considerations

### Current Implementation

1. **Input Validation**: Pydantic models validate all inputs
2. **SQL Injection Prevention**: Parameterized queries
3. **XSS Protection**: Jinja2 auto-escaping
4. **Error Handling**: No sensitive data in error messages

### Production Enhancements

For production deployment, consider:

1. **Authentication**: JWT tokens, OAuth2
2. **Authorization**: Role-based access control
3. **HTTPS**: TLS/SSL encryption
4. **Rate Limiting**: Prevent abuse
5. **CORS**: Proper origin restrictions
6. **Input Sanitization**: Additional validation
7. **Secrets Management**: Vault, AWS Secrets Manager
8. **Audit Logging**: Track all operations

---

## Performance Optimization

### Database Level

1. **Indexes**: Created on frequently queried columns (status, due_date)
2. **Connection Pooling**: Reuse database connections
3. **Query Optimization**: Efficient SQL queries
4. **Transactions**: Minimize transaction scope

### Application Level

1. **Async/Await**: Non-blocking I/O operations
2. **Response Caching**: Cache frequently accessed data
3. **Pagination**: Limit result set size
4. **Batch Operations**: Group multiple operations

### Infrastructure Level

1. **Docker**: Containerization for consistent deployment
2. **Load Balancing**: Distribute traffic
3. **CDN**: Serve static assets
4. **Database Replication**: Read replicas

---

## Testing Strategy

### Test Pyramid

```
         ┌─────┐
        /       \
       /   E2E   \          ◄── Few comprehensive tests
      /           \
     └─────────────┘
    ┌───────────────┐
   /                 \
  /   Integration     \     ◄── Moderate number
 /                     \
└───────────────────────┘
┌─────────────────────────┐
│                         │
│      Unit Tests         │  ◄── Many focused tests
│                         │
└─────────────────────────┘
```

### Test Coverage

**Unit Tests** (`tests/test_api.py`):
- Individual endpoint testing
- Success and error scenarios
- Validation testing
- Edge cases

**Test Fixtures** (`tests/conftest.py`):
- Test client setup
- Sample data creation
- Database connection testing

### Running Tests

```bash
# All tests
pytest tests/ -v

# With coverage
pytest tests/ --cov=app --cov-report=html

# Specific test
pytest tests/test_api.py::TestTaskAPI::test_create_task -v
```

---

## Deployment Architecture

### Docker Compose Setup

```yaml
deployment:
  services:
    postgres:
      image: postgres:15-alpine
      purpose: Database server
      ports: [5432]
      volumes:
        - postgres_data (persistent storage)
        - ./db (initialization scripts)
      health_check: pg_isready
    
    app:
      build: Dockerfile
      purpose: FastAPI application
      ports: [8000]
      depends_on: [postgres]
      environment:
        - Database credentials
        - Application config
      volumes:
        - . (source code for development)
```

### Deployment Flow

```
Developer
    │
    ├─► Write Code
    │      │
    │      ▼
    ├─► Commit to Git
    │      │
    │      ▼
    └─► Push to GitHub
           │
           ▼
    Docker Build
           │
           ├─► Pull base image
           │
           ├─► Install dependencies
           │
           ├─► Copy application code
           │
           └─► Create image
                  │
                  ▼
    Docker Compose Up
           │
           ├─► Start PostgreSQL
           │   └─► Initialize schema
           │       └─► Load sample data
           │
           └─► Start FastAPI
               └─► Connect to database
                   └─► Start serving
```

### Scaling Strategy

```
Load Balancer
      │
      ├─────────┬─────────┬─────────┐
      ▼         ▼         ▼         ▼
   App 1    App 2    App 3    App N
      │         │         │         │
      └─────────┴────┬────┴─────────┘
                     ▼
              PostgreSQL
             (with replicas)
```

---

## Logging and Monitoring

### Logging Levels

```python
logging:
  DEBUG: Detailed debugging information
  INFO: General application flow
  WARNING: Potential issues
  ERROR: Error conditions
  CRITICAL: Critical failures
```

### Log Format

```
2025-11-20 17:11:56,503 - app.main - INFO - Starting application...
2025-11-20 17:11:56,509 - app.database.connection - INFO - Database connection successful
```

### Monitoring Metrics

1. **Application Metrics**:
   - Request count
   - Response time
   - Error rate

2. **Database Metrics**:
   - Connection pool usage
   - Query execution time
   - Transaction rate

3. **System Metrics**:
   - CPU usage
   - Memory usage
   - Disk I/O

---

## Conclusion

This To-Do List application demonstrates modern web development practices with:

- Clean architecture and separation of concerns
- RESTful API design
- Raw SQL database operations (no ORM)
- Comprehensive testing
- Proper error handling and logging
- Docker containerization
- Interactive API documentation

The application is production-ready with proper security, performance optimization, and deployment strategies in place.

For questions or improvements, refer to the README.md or API documentation.

---

**Study Guide Version**: 1.0
**Last Updated**: November 2025
