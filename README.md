# To-Do List Application with FastAPI

A comprehensive To-Do List management application built with FastAPI, PostgreSQL, and HTML templates. This project demonstrates RESTful API design, database management without ORM, and modern web development practices.

## 🌟 Features

- **RESTful API**: Full CRUD operations for task management
- **Database**: PostgreSQL with raw SQL queries (no ORM)
- **Web Interface**: Responsive HTML templates with modern UI
- **API Documentation**: Interactive Swagger UI and ReDoc
- **Testing**: Comprehensive test suite with pytest
- **Logging**: Structured logging throughout the application
- **Exception Handling**: Robust error handling and validation
- **Docker Support**: Containerized deployment with Docker Compose
- **Sample Data**: Pre-loaded example tasks for demonstration

## 📸 Screenshots

### Homepage - Task List
![Homepage](https://github.com/user-attachments/assets/b966f0f8-bcdb-46be-9a3d-f3dd841eee02)

### Add New Task
![Add Task](https://github.com/user-attachments/assets/04852ffd-f6f8-49cf-8f53-7b10e8fcbcd3)

## 🏗️ Architecture

The application follows a layered architecture:

```
├── app/                      # Application code
│   ├── api/                  # API endpoints
│   ├── database/             # Database layer (raw SQL)
│   ├── templates/            # HTML templates
│   ├── static/               # Static files (CSS)
│   ├── config.py             # Configuration management
│   ├── models.py             # Pydantic models
│   └── main.py               # FastAPI application
├── db/                       # Database files
│   ├── init.sql              # Schema initialization
│   ├── sample_data.sql       # Sample data
│   └── backup.sql            # Database backup
├── tests/                    # Test suite
├── study/                    # Documentation
├── docker-compose.yml        # Docker Compose configuration
├── Dockerfile                # Docker image
├── requirements.txt          # Python dependencies
└── .env                      # Environment variables
```

## 🚀 Getting Started

### Prerequisites

- Docker and Docker Compose
- Python 3.11+ (for local development)
- PostgreSQL 15+ (if running without Docker)

### Quick Start with Docker

1. **Clone the repository**
```bash
git clone https://github.com/aasthatayal/Pelocal_ass.git
cd Pelocal_ass
```

2. **Start the application**
```bash
docker compose up -d
```

3. **Access the application**
- Web Interface: http://localhost:8000
- API Documentation: http://localhost:8000/docs
- Alternative API Docs: http://localhost:8000/redoc

4. **Stop the application**
```bash
docker compose down
```

### Local Development Setup

1. **Install Python dependencies**
```bash
pip install -r requirements.txt
```

2. **Start PostgreSQL**
```bash
docker compose up -d postgres
```

3. **Configure environment**
```bash
# .env file is already configured for local development
# Modify if needed
```

4. **Run the application**
```bash
python -m app.main
```

5. **Run tests**
```bash
pytest tests/ -v
```

## 📊 Database

### Schema

The application uses a single `tasks` table:

```sql
CREATE TABLE tasks (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    due_date DATE,
    status VARCHAR(50) DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Restore from Backup

To restore the database from the backup file:

```bash
# Method 1: Using docker compose
docker compose exec -T postgres psql -U todo_user -d todo_db < db/backup.sql

# Method 2: Using docker directly
docker exec -i todo_postgres psql -U todo_user -d todo_db < db/backup.sql

# Method 3: Copy file into container and restore
docker cp db/backup.sql todo_postgres:/tmp/backup.sql
docker compose exec postgres psql -U todo_user -d todo_db -f /tmp/backup.sql
```

### Create New Backup

```bash
docker compose exec -T postgres pg_dump -U todo_user -d todo_db > db/backup_$(date +%Y%m%d_%H%M%S).sql
```

## 🔌 API Endpoints

### Task Operations

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/tasks` | Get all tasks (with optional status filter) |
| GET | `/api/tasks/{id}` | Get a specific task |
| POST | `/api/tasks` | Create a new task |
| PUT | `/api/tasks/{id}` | Update a task |
| DELETE | `/api/tasks/{id}` | Delete a task |

### System Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/health` | Health check |
| GET | `/docs` | Swagger UI documentation |
| GET | `/redoc` | ReDoc documentation |

### Example API Usage

**Create a task:**
```bash
curl -X POST http://localhost:8000/api/tasks \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Learn FastAPI",
    "description": "Complete FastAPI tutorial",
    "due_date": "2025-12-31",
    "status": "pending"
  }'
```

**Get all tasks:**
```bash
curl http://localhost:8000/api/tasks
```

**Filter by status:**
```bash
curl http://localhost:8000/api/tasks?status=pending
```

**Update a task:**
```bash
curl -X PUT http://localhost:8000/api/tasks/1 \
  -H "Content-Type: application/json" \
  -d '{
    "status": "completed"
  }'
```

**Delete a task:**
```bash
curl -X DELETE http://localhost:8000/api/tasks/1
```

## 🧪 Testing

The project includes comprehensive tests for all API endpoints:

```bash
# Run all tests
pytest tests/ -v

# Run specific test file
pytest tests/test_api.py -v

# Run with coverage
pytest tests/ --cov=app --cov-report=html
```

### Test Coverage

- ✅ Health check endpoint
- ✅ Create task (valid and invalid inputs)
- ✅ Get all tasks
- ✅ Get tasks by status
- ✅ Get single task
- ✅ Update task (full and partial)
- ✅ Delete task
- ✅ Complete task lifecycle
- ✅ Error handling and validation

## 📝 Logging

The application implements comprehensive logging:

- **INFO**: General application flow
- **WARNING**: Potential issues
- **ERROR**: Error conditions
- **DEBUG**: Detailed debugging information

Logs include:
- Database operations
- API requests and responses
- Validation errors
- Exception details

## 🔒 Exception Handling

Robust exception handling throughout:

- **Database errors**: Connection issues, query failures
- **Validation errors**: Invalid input data
- **Not found errors**: Missing resources
- **Internal server errors**: Unexpected exceptions

All errors return appropriate HTTP status codes and detailed error messages.

## 🛠️ Configuration

Environment variables (`.env` file):

```env
# Database Configuration
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
POSTGRES_DB=todo_db
POSTGRES_USER=todo_user
POSTGRES_PASSWORD=todo_password

# Application Configuration
APP_HOST=0.0.0.0
APP_PORT=8000
APP_RELOAD=True

# Logging Configuration
LOG_LEVEL=INFO
```

## 📚 Documentation

Detailed documentation is available in the `study/` directory:

- **study.md**: Architecture overview, design decisions, and diagrams
- **API Documentation**: Available at `/docs` and `/redoc` when running

## 🎨 Technologies Used

- **FastAPI**: Modern Python web framework
- **PostgreSQL**: Relational database
- **psycopg2**: PostgreSQL adapter (raw SQL)
- **Pydantic**: Data validation
- **Jinja2**: Template engine
- **pytest**: Testing framework
- **Docker**: Containerization
- **Uvicorn**: ASGI server

## 📋 Requirements

See `requirements.txt` for complete list of Python dependencies.

## 🤝 Contributing

This is an assignment project. For production use, consider:

- Adding authentication and authorization
- Implementing pagination for large datasets
- Adding rate limiting
- Implementing caching
- Adding more comprehensive error handling
- Implementing task categories and tags
- Adding user management

## 📄 License

This project is created as an assignment for educational purposes.

## 👨‍💻 Author

Created as part of the Pelocal Assignment.

## 🙏 Acknowledgments

- FastAPI framework and documentation
- PostgreSQL community
- Python community

## 📞 Support

For issues or questions, please refer to the API documentation at `/docs` or `/redoc`.

---

**Built with ❤️ using FastAPI & PostgreSQL**
