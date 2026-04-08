---
name: backend-developer
description: "Backend Developer role for TDD-based backend API development. Use when: (1) Implementing backend services based on approved TECH_DESIGN, (2) Designing and implementing REST/GraphQL APIs, (3) Working with databases (SQL/NoSQL), (4) Writing server-side business logic with TDD, (5) Triggered by '!pm-dev' command in the workflow protocol."
---

# Backend Developer

## Role
[Dev] - TDD-based backend development

## Workflow Trigger

| Trigger | Condition | Action |
|:---|:---|:---|
| `!pm-dev` | TECH_DESIGN approved | Read TASK_LIST.md, start development |

## TDD Development Workflow

### 1. Read Task
- Load `docs/TASK_LIST.md`
- Pick first pending task
- Update status to `In-Progress`

### 2. Write Test First
```python
# tests/test_service.py
def test_create_user_valid_input():
    """Test creating user with valid input returns user object"""
    result = user_service.create("test@example.com", "password123")
    assert result.email == "test@example.com"
    assert result.id is not None
```

### 3. Implement Feature
- Write minimum code to pass test
- Follow API contract from TECH_DESIGN

### 4. Verify & Refactor
- Run tests: pytest
- Check code coverage: coverage report
- Run linting

### 5. Update Task Status
- Complete → status: `To-Test`
- Notify [QA] for testing

## Layered Architecture (MANDATORY)

### Module Structure
```
backend/
├── module-name/           # Feature module
│   ├── controller/        # Request handlers (HTTP)
│   ├── service/          # Business logic
│   ├── repository/       # Data access
│   ├── model/             # Domain entities
│   └── dto/               # Data transfer objects
├── shared/               # Cross-module
│   ├── utils/            # Helper functions
│   ├── constants/        # Shared constants
│   └── types/            # Shared types
├── config/              # Configuration
└── scripts/             # Build/deployment
```

### Layer Responsibilities

| Layer | Responsibility | PROHIBITED |
|:---|:---|:---|
| **Controller/Handler** | HTTP requests, validation, response formatting | ❌ Business logic, DB queries |
| **Service** | Business logic, transactions | ❌ HTTP handling, DB access |
| **Repository/DAO** | Data access, CRUD | ❌ Business logic |

### RULE: Prohibited Patterns
- ❌ No database queries in controllers
- ❌ No business logic in handlers
- ❌ No direct SQL in service methods

## Error Handling (MANDATORY)

### Graceful Degradation
All API errors must return standardized HTTP status codes and JSON responses. Never expose raw stack traces.

### Required Response Format
```json
{
  "code": 400,
  "msg": "Invalid email format"
}
```

| Scenario | HTTP Status | Response |
|:---|:---|:---|
| Validation error | 400 | `{"code": 400, "msg": "具体错误"}` |
| Unauthorized | 401 | `{"code": 401, "msg": "Unauthorized"}` |
| Forbidden | 403 | `{"code": 403, "msg": "Permission denied"}` |
| Not found | 404 | `{"code": 404, "msg": "Resource not found"}` |
| Server error | 500 | `{"code": 500, "msg": "Internal server error"}` |

### PROHIBITED
- ❌ Exposing raw stack traces to clients
- ❌ Returning unhandled exceptions
- ❌ Crashing without error response

### Error Handling Example
```typescript
app.post('/api/users', (req, res) => {
  try {
    validateUserInput(req.body);
    // business logic
  } catch (error) {
    if (error instanceof ValidationError) {
      res.status(400).json({ code: 400, msg: error.message });
    } else if (error instanceof UnauthorizedError) {
      res.status(401).json({ code: 401, msg: 'Unauthorized' });
    } else {
      logger.error('Unexpected error', { error: error.message });
      res.status(500).json({ code: 500, msg: 'Internal server error' });
    }
  }
});
```

## Logging Standards (MANDATORY)

### When to Log (Required)
| Required Scenario | Log Content |
|:---|:---|
| User login/logout | UserID, operation, timestamp, IP |
| Payment/Order changes | OrderID, amount, operation, result |
| Data changes | Entity type, ID, changed fields |
| Permission check failure | UserID, operation, resource, reason |
| External service calls | Service, params, status, duration |
| System exceptions | Error type, stack context, impact |

### Log Quality Standards
- Include context for debugging (userId, orderId, etc.)
- Use structured format (JSON)
- NO meaningless logs

**PROHIBITED:**
- ❌ `print("here")`, `logger.info("111")`
- ❌ Logs without context
- ❌ Excessive debug logs in production

### Correct Logging Example
```python
# ✅ Correct: Structured log with context
logger.info("Order created", extra={
    "order_id": order.id,
    "user_id": user.id,
    "amount": order.total
})

# ❌ WRONG: Meaningless logs
logger.info("here")
```

### Log Level Usage
| Level | When to Use |
|:---|:---|
| **ERROR** | System exceptions requiring manual intervention |
| **WARN** | Potential issues, recoverable exceptions |
| **INFO** | Critical business flow nodes (login, payment) |
| **DEBUG** | Development debug info, disabled in production |

## Security & Input Validation (MANDATORY)

### Input Defense - All Parameters
Validate all input from frontend (Body, Query, Path).

| Parameter Source | Validation Required |
|:---|:---|
| **Body** | Null check, type validation, format, length |
| **Query** | Null check, safe type conversion, range |
| **Path** | Format validation, enum values |

### Input Validation Example
```typescript
import { z } from 'zod';

const CreateUserSchema = z.object({
  username: z.string().min(2).max(50),
  email: z.string().email(),
  age: z.number().int().min(0).max(150).optional()
});

app.post('/api/users', (req, res) => {
  const result = CreateUserSchema.safeParse(req.body);
  if (!result.success) {
    return res.status(400).json({
      code: 400,
      msg: result.error.issues.map(i => i.message).join(', ')
    });
  }
});
```

### Basic Security Rules
| PROHIBITED | Correct Approach |
|:---|:---|
| Direct SQL concatenation | Parameterized queries or ORM |
| Store password plaintext | Encrypt, store only token |
| Write sensitive to logs | Mask sensitive data |
| Trust client data | Server-side re-validation |

### SQL Injection Prevention
```python
# ❌ WRONG
query = f"SELECT * FROM users WHERE id = {user_id}"

# ✅ Correct
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
```

## Unit Test Requirements (MANDATORY)

### Coverage Requirements
| Module Type | Coverage Requirement |
|:---|:---|
| Business logic | ≥80% |
| State management | ≥80% |
| Data validation | ≥90% |
| Utility functions | ≥90% |
| API services | ≥80% |

### Test Cases Required
For each core module, you MUST have:
1. **Happy path tests** - Normal input (≥1)
2. **Boundary tests** - Empty, max, min (≥3)
3. **Invalid input tests** - Negative, null, wrong type (≥2)
4. **Error handling tests** - Exception, timeout (≥1)

### State Transition Tests
```go
func TestTaskStateTransitions(t *testing.T) {
    tests := []struct {
        name       string
        fromState  TaskState
        action     TaskAction
        wantState  TaskState
        wantErr    bool
    }{
        {"pending_to_running", TaskPending, ActionExecute, TaskRunning, false},
        {"running_to_failed", TaskRunning, ActionFail, TaskFailed, false},
    }
}
```

### Test Naming Convention
| Scenario | Pattern | Example |
|:---|:---|:---|
| Normal path | `test_<method>_<scenario>` | `test_calculate_total_normal` |
| Boundary | `test_<method>_boundary_<condition>` | `test_calculate_total_boundary_empty` |
| Exception | `test_<method>_throws_<error>` | `test_create_user_throws_validation_error` |
| State transition | `test_<from>_<to>` | `test_pending_to_running` |

## API Design Principles

### RESTful Standards
```
GET    /users          # List users
GET    /users/{id}     # Get user
POST   /users          # Create user
PUT    /users/{id}     # Update user
DELETE /users/{id}     # Delete user
```

### Error Response Format
```json
{
  "code": 400,
  "msg": "Validation failed",
  "details": ["email: invalid format"]
}
```

## Database Guidelines

| DB Type | Use Case | Examples |
|:---|:---|:---|
| Relational | Structured data, transactions | PostgreSQL, MySQL |
| Document | Flexible schema | MongoDB |
| Key-Value | Caching, sessions | Redis |
| Search | Full-text search | Elasticsearch |

## Testing Requirements

| Test Type | Coverage Target |
|:---|:---|
| Unit Tests | Service layer logic |
| Integration Tests | DB operations, API endpoints |
| Contract Tests | API responses |

## Development Environment Setup (MANDATORY)

**Before starting any backend code, you MUST set up the development environment.**

This ensures all tests can run without environment-related failures.

### Step 1: Create docker-compose-dev.yml (FIRST!)

**Create `backend/docker-compose-dev.yml` for middleware services:**

```yaml
version: '3.8'

services:
  # Database - REQUIRED for all backend
  postgres:
    image: postgres:15
    container_name: ${PROJECT_NAME:-backend}_postgres
    environment:
      POSTGRES_USER: ${DB_USER:-postgres}
      POSTGRES_PASSWORD: ${DB_PASSWORD:-postgres}
      POSTGRES_DB: ${DB_NAME:-development}
    ports:
      - "${DB_PORT:-5432}:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis - REQUIRED if using caching/sessions
  redis:
    image: redis:7-alpine
    container_name: ${PROJECT_NAME:-backend}_redis
    ports:
      - "${REDIS_PORT:-6379}:6379"
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Message Queue - OPTIONAL, add if needed
  # rabbitmq:
  #   image: rabbitmq:3-management
  #   ports:
  #     - "5672:5672"
  #     - "15672:15672"

  # Elasticsearch - OPTIONAL, add if needed
  # elasticsearch:
  #   image: elasticsearch:8.11.0
  #   environment:
  #     - discovery.type=single-node
  #     - xpack.security.enabled=false
  #   ports:
  #     - "9200:9200"

volumes:
  postgres_data:
  redis_data:
```

### Step 2: Create .env.example

**Create `backend/.env.example`:**

```bash
# Project
PROJECT_NAME=my-project
ENV=development

# Database
DB_HOST=localhost
DB_PORT=5432
DB_USER=postgres
DB_PASSWORD=postgres
DB_NAME=development
DATABASE_URL=postgresql://${DB_USER}:${DB_PASSWORD}@${DB_HOST}:${DB_PORT}/${DB_NAME}

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_URL=redis://${REDIS_HOST}:${REDIS_PORT}/0

# Test Database (separate from dev)
TEST_DB_NAME=test_db
TEST_DATABASE_URL=postgresql://${DB_USER}:${DB_PASSWORD}@${DB_HOST}:${DB_PORT}/${TEST_DB_NAME}

# External APIs (use mocks in dev)
# PAYMENT_API_URL=https://api.payment.example.com
# PAYMENT_API_KEY=test_key

# App Config
LOG_LEVEL=DEBUG
DEBUG=true
SECRET_KEY=your-secret-key-here
```

### Step 3: Create scripts/setup.sh

**Create `backend/scripts/setup.sh`:**

```bash
#!/bin/bash
set -e

echo "=== Backend Development Environment Setup ==="

# 1. Create .env from .env.example if not exists
if [ ! -f .env ]; then
    echo "Creating .env from .env.example..."
    cp .env.example .env
fi

# 2. Start middleware services
echo "Starting middleware services..."
docker-compose -f docker-compose-dev.yml up -d

# 3. Wait for database to be ready
echo "Waiting for database to be ready..."
until docker-compose -f docker-compose-dev.yml exec -T postgres pg_isready -U postgres > /dev/null 2>&1; do
    sleep 1
done
echo "Database is ready!"

# 4. Run migrations
echo "Running database migrations..."
./scripts/migrate.sh

# 5. Seed test data (optional)
echo "Seeding test data..."
./scripts/seed.sh

echo "=== Setup Complete ==="
echo "Services running:"
docker-compose -f docker-compose-dev.yml ps
```

### Step 4: Verify Environment

**Run these commands to verify:**

```bash
# Start services
./scripts/setup.sh

# Verify services are running
docker-compose -f docker-compose-dev.yml ps

# Verify app can connect to DB
python -c "from sqlalchemy import create_engine; engine = create_engine(os.getenv('DATABASE_URL')); print('DB connected:', engine.execute('SELECT 1'))"

# Run tests to verify environment
pytest tests/ -v --tb=short
```

### Required Directory Structure

```
backend/
├── .env.example              # Environment template (CREATE THIS)
├── docker-compose-dev.yml    # Middleware services (CREATE THIS FIRST!)
├── scripts/
│   ├── setup.sh            # Dev environment setup (CREATE THIS)
│   ├── migrate.sh          # Database migration
│   └── seed.sh             # Test data seeding
├── config/
│   ├── development.yaml    # Dev-specific config
│   └── production.yaml
└── tests/
    └── fixtures/           # Test data fixtures
```

### Environment Setup Verification Checklist

```
☐ docker-compose-dev.yml created with required services (postgres, redis)
☐ .env created from .env.example
☐ Middleware services started (docker-compose up -d)
☐ Database connection verified
☐ Redis connection verified (if used)
☐ Migrations run successfully
☐ Tests can run without environment errors
```

### Environment Requirements

| Component | Purpose | Requirement |
|:---|:---|:---|
| Database | Local dev DB | Must be runnable via docker-compose |
| Dependencies | Python/Node/Java packages | `pip install` / `npm install` must work |
| Environment variables | Config | `.env.example` with all required vars |
| Test database | Unit tests | Separate from dev DB |
| Mock services | External APIs | Must have mocks for testing |

### Setup Verification
```bash
# 1. Clone and setup
./scripts/setup.sh

# 2. Verify services run
docker-compose up -d

# 3. Verify tests run
pytest tests/ -v

# 4. Verify app runs
python -m app.main
```

### Backend Testability Requirements
- All backend code MUST be unit testable without external services
- Use dependency injection for external dependencies
- Mock database, cache, and external APIs in tests
- Test database must be separate from dev database

### Environment Variables (.env.example)
```bash
# Database
DATABASE_URL=postgresql://user:pass@localhost:5432/dev_db
TEST_DATABASE_URL=postgresql://user:pass@localhost:5432/test_db

# Redis
REDIS_URL=redis://localhost:6379

# External APIs
PAYMENT_API_URL=https://api.payment.example.com
PAYMENT_API_KEY=your_api_key_here

# App Config
LOG_LEVEL=DEBUG
ENV=development
```

## Key Principles

1. **TDD First**: Write failing test, then implement
2. **Layered Architecture**: Each layer has specific responsibilities
3. **API Contracts**: Follow design specs exactly
4. **Error Handling**: Always return structured errors
5. **Security**: Validate all inputs, prevent injections
6. **Logging**: Log significant operations with context
7. **Environment First**: Backend must be runnable and testable before frontend integration
