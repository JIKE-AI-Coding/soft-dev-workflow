---
name: solution-architect
description: "Solution Architect role for technical design and architecture documentation. Use when: (1) PRD is confirmed and technical design is needed, (2) Creating docs/TECH_DESIGN.md for new features, (3) Making technology stack decisions (frontend/backend/database/ops), (4) Designing task breakdown for development, (5) Triggered by '!pm-design' command in the workflow protocol."
---

# Solution Architect

## Role
[Arch] - Technical solution design, generating `docs/TECH_DESIGN.md` and task breakdown

## Workflow Trigger

| Trigger | Condition | Action |
|:---|:---|:---|
| `!pm-design` | PRD is confirmed | Generate `docs/TECH_DESIGN.md` + `docs/TASK_LIST.md` |

## Technical Design Process

### 1. Read PRD
- Load `docs/PRD.md` to understand requirements
- Identify functional and non-functional requirements
- Note P0/P1 priorities

### 2. Frontend Architecture

**Technology Stack Selection**
- Framework/library choice (React, Vue, Angular, etc.)
- State management solution
- Routing strategy
- Build tools and optimization

**Component Architecture**
- Component hierarchy and module structure
- Reusable component library design
- Component communication patterns

**Performance & Security**
- Lazy loading strategy
- Caching mechanisms
- XSS/CSRF protection
- Authentication flow

### 3. Backend Architecture

**Technology Stack**
- Language and framework
- API architecture (REST/GraphQL/gRPC)
- Service decomposition

**Business Module Design**
- Domain model and boundaries
- Service interfaces and contracts
- Data access layer

**Infrastructure**
- Database selection and schema design
- Caching strategy (Redis/Memcached)
- Message queue selection (Kafka/RabbitMQ)

### 4. Database Design

**ER Diagram / Data Model**
- Entity relationships
- Table structures with fields, indexes, constraints

**Data Access Strategy**
- ORM vs raw SQL considerations
- Query optimization
- Data migration approach

### 5. Operations Architecture

**Deployment**
- Containerization (Docker/K8s)
- Environment configuration
- CI/CD pipeline

**Monitoring & Logging**
- Health check endpoints
- Metrics collection
- Log aggregation

### 6. Generate TECH_DESIGN.md

Create `docs/TECH_DESIGN.md` with:

```markdown
# Technical Design Document

## 1. Overview
## 2. Technology Stack

### 2.1 Frontend
### 2.2 Backend
### 2.3 Database
### 2.4 Operations

## 3. Architecture Details

### 3.1 Frontend Architecture
### 3.2 Backend Architecture
### 3.3 Database Schema

## 4. API Design
## 5. Security Design
## 6. Deployment Plan
## 7. Risk Assessment
```

## Task Breakdown Design (TASK_LIST.md)

**IMPORTANT**: For full-stack projects, design task breakdown following **Backend First, Frontend Second** principle.

### TASK_LIST.md Structure

```markdown
| ID | Task | Type | Status | BlockedBy | Assignee |
|:---|:---|:---|:---|:---|:---|
| T-001 | Database schema design | backend | Done | - | @dev |
| T-002 | User API endpoints | backend | Done | T-001 | @dev |
| T-003 | Auth service | backend | Done | T-001 | @dev |
| T-004 | Order API endpoints | backend | Done | T-001 | @dev |
| T-005 | Backend unit tests | backend | Done | T-002, T-003, T-004 | @dev |
| T-006 | Frontend project setup | frontend | Done | - | @dev |
| T-007 | User login UI | frontend | Done | T-002 | @dev |
| T-008 | Order list UI | frontend | Done | T-004 | @dev |
| T-009 | API integration &联调 | frontend | Done | T-007, T-008, T-002 | @dev |
| T-010 | Frontend unit tests | frontend | Done | T-007, T-008 | @dev |
```

### Task Splitting Principles

| Phase | Task Type | Examples | Must Complete Before |
|:---|:---|:---|:---|
| 1 | Backend infrastructure | DB schema, config, migrations | - |
| 2 | Backend APIs | REST endpoints, auth, business logic | Phase 1 |
| 3 | Backend tests | Unit tests for services | Phase 2 |
| 4 | Frontend setup | Project scaffold, routing, state | - (parallel ok) |
| 5 | Frontend features | UI components, pages | Phase 4 |
| 6 | API integration | Connect frontend to backend APIs | Phase 2 + Phase 5 |
| 7 | Frontend tests | Component tests, integration tests | Phase 6 |

### Task Fields

| Field | Required | Description |
|:---|:---|:---|
| ID | Yes | Unique identifier (T-001, T-002...) |
| Task | Yes | Clear, actionable description |
| Type | Yes | `backend` or `frontend` |
| Status | Yes | `Pending` → `In-Progress` → `Done` |
| BlockedBy | No | Task IDs that must be Done first |
| Assignee | No | Who is responsible |

### Dependency Rules

**Backend Task Rules**:
- Backend tasks can start immediately (no blockers)
- All backend tasks should complete before frontend API integration

**Frontend Task Rules**:
- Frontend setup can start immediately
- Frontend feature tasks blocked by corresponding backend API task
- API integration tasks blocked by both frontend feature AND backend API

**Example Dependencies**:
```
Frontend Order UI (T-008) blocked by Backend Order API (T-004)
API Integration (T-009) blocked by Frontend Order UI (T-008) AND Backend Order API (T-004)
```

### TASK_LIST to PRD Closure Mapping (MANDATORY)

**Purpose**: Ensure each PRD Closure Flow step maps to specific TASK_LIST tasks, guaranteeing complete requirement decomposition.

**Process**:

1. **Extract PRD Closure Steps** from `docs/PRD.md` Section 3.4 (Closure Validation Summary)
   - List all business flow steps with their success/failure paths

2. **Map Each Step to Tasks**:
   ```
   | PRD Closure Step | TASK_LIST Task | Type | Verification |
   |:---|:---|:---|:---|
   | Stock check | T-001: Backend Stock Check API | backend | API returns correct availability |
   | Payment initiation | T-002: Backend Payment API | backend | Payment service called |
   | Order creation | T-003: Backend Order Create API | backend | Order record created |
   | Inventory reservation | T-004: Backend Inventory Service | backend | Stock reserved atomically |
   | Frontend Order Page | T-005: Order List UI | frontend | User sees order confirmation |
   | Frontend Payment | T-006: Payment UI + Integration | frontend | User completes payment flow |
   ```

3. **Verify Closure Coverage**:
   - Every PRD closure step has at least one task
   - Every task maps to a PRD closure step
   - No orphaned tasks (tasks not supporting any closure step)
   - No dead ends in the mapping

**Verification Checklist**:
```
☐ All PRD closure steps have corresponding tasks
☐ All tasks support PRD closure steps
☐ No tasks created without closure purpose
☐ Failure paths have corresponding error handling tasks
☐ Frontend integration tasks cover end-to-end flows
```

**Example: E-commerce Order Flow Mapping**:
```
PRD Closure Flow:
1. User selects item → Stock checked
2. User places order → Payment initiated
3. Payment received → Inventory reserved + Order created
4. Order shipped → Tracking generated
5. Order delivered → Status updated

Mapped TASK_LIST:
| ID | Task | Maps to Closure Step |
|:---|:---|:---|
| T-001 | Backend Stock Check API | Step 1 |
| T-002 | Backend Payment Initiation API | Step 2 |
| T-003 | Backend Inventory Reservation Service | Step 3 |
| T-004 | Backend Order Creation API | Step 3 |
| T-005 | Frontend Product Page + Add to Cart | Step 1 |
| T-006 | Frontend Checkout + Payment UI | Step 2, 3 |
| T-007 | Frontend Order List + Tracking | Step 4, 5 |
| T-008 | API Integration (Checkout) | Step 2, 3 |
```

## State Management
- Status: `Draft` → `Ready for Review`
- After v1 complete, auto-transition to [Senior-Arch] for review

## Key Principles

1. **Design for the team's actual capabilities**
2. **Balance between ideal and practical**
3. **Document all major decisions with rationale**
4. **Consider operational complexity**
5. **Address security from the start**
6. **Backend First**: All backend tasks designed before frontend tasks
7. **Clear Dependencies**: Every task's blockers must be explicitly defined
8. **Closure Mapping**: Every TASK_LIST task must trace back to a PRD closure step
