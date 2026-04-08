---
name: development-workflow
description: "Complete development workflow orchestration skill that coordinates the full software development lifecycle from requirements to deployment. Use when: (1) Starting a new project or feature, (2) Need to execute the complete workflow cycle (!pm-analyze, !pm-design, !pm-review, !pm-dev, !pm-test, !pm-deploy, !pm-uat), (3) Managing a sprint or iteration, (4) Coordinating multiple team roles (PM/Arch/Dev/QA/DevOps). **IMPORTANT**: When executing phases, you MUST use the `Skill` tool to invoke the corresponding role skill. This is a mandatory requirement, not optional."
---

# Development Workflow Orchestrator

## Role
Orchestrates the complete software development lifecycle by invoking individual role skills in proper sequence.

## Important: Skill Invocation Requirement

**MANDATORY**: When executing any phase, you MUST use the `Skill` tool to invoke the corresponding skill:
```
Skill: skill-name
Example: Skill: product-manager
```

Do NOT skip this step. The workflow orchestrator delegates to specialized skills.

## Workflow Pipeline

```
[PM] --> [Arch] --> [Senior-Arch] --> [Dev] --> [QA] --> [DevOps] --> [PM]
  ↓        ↓            ↓              ↓         ↓          ↓         ↓
!pm-analyze !pm-design !pm-review    !pm-dev   !pm-test  !pm-deploy !pm-uat
                                    ↗                                      ↙
                                (bug found)                              (fail)
                                    ↖                                      ↙
                                  [!pm-fix] <--- [Hotfix for production]
```

## Trigger Commands

| Command | Role Invoked | Condition | Skill to Invoke |
|:---|:---|:---|:---|
| `!pm-analyze` | [PM] | User proposes new requirements | `product-manager` |
| `!pm-design` | [Arch] | PRD confirmed | `solution-architect` |
| `!pm-review` | [Senior-Arch] | TECH_DESIGN generated | `senior-architect` |
| `!pm-dev` | [Dev] | TECH_DESIGN approved + TASK_LIST exists | `frontend-developer` or `backend-developer` |
| `!pm-test` | [QA] | Code deployed to test | `qa-engineer` |
| `!pm-fix` | [Dev] | QA finds bugs (from !pm-test loop) | `frontend-developer` or `backend-developer` |
| `!pm-deploy` | [DevOps] | QA testing passed | `devops-engineer` |
| `!pm-uat` | [PM] | Code deployed to staging | `product-manager` |
| `!pm-hotfix` | [Dev] | Production bug reported | `backend-developer` or `frontend-developer` |

## Phase 1: Requirements (!pm-analyze)

**Role**: [PM]

**MANDATORY**: `Skill: product-manager`

**Purpose**: Analyze requirements, generate PRD with hidden requirements, implicit constraints, and **closure validation**.

### Steps

1. Read user requirements
2. Analyze hidden requirements
3. Identify implicit constraints (business closure, logic self-consistency)
4. **PRD Closure Validation** (MUST pass before proceeding)
5. Generate PRD to `docs/PRD.md`
6. Set status: `Draft` → `Confirmed`

### PRD Closure Validation (MANDATORY)

**Before generating PRD, PM MUST verify:**

| Check | Description | Pass Criteria |
|:---|:---|:---|
| **Business Closure** | Each feature has complete start→end flow | No dead ends |
| **Data Integrity** | Data consistency maintained across operations | No data loss/corruption |
| **State Validity** | All state transitions are valid | No invalid states possible |
| **Error Recovery** | Failures have recovery paths | No unrecoverable states |
| **Integration Points** | External dependencies are identified | No missing integrations |

**Validation Rules:**

```
❌ INVALID PRD Examples:
- "User can place order" → No mention of inventory check
- "User can book slot" → No mention of duplicate booking prevention
- "Order placed successfully" → No definition of what "success" means

✅ VALID PRD Examples:
- "User can place order IF stock ≥ quantity → inventory reserved atomically"
- "User can book slot IF slot not already booked → booking confirmed"
- "Order placed = payment received + inventory reserved + order created in DB"
```

### PRD Closure Checklist

```
PRD Closure Validation Checklist:
☐ Business flows have clear start and end points
☐ Every action has a defined outcome
☐ Data produced by one step is available to next step
☐ Failure scenarios have defined recovery paths
☐ External system dependencies are identified
☐ No circular dependencies in flows
☐ State transitions are explicitly defined
☐ Data ownership is clear (who creates, updates, deletes)
```

**If validation FAILS**: PRD is incomplete → Return to analysis → Fix before proceeding

**Output**: `docs/PRD.md` (with closure validation passed)

## Phase 2: Technical Design (!pm-design)

**Role**: [Arch]

**MANDATORY**: `Skill: solution-architect`

**Purpose**: Design system architecture including:
- Frontend architecture
- Backend architecture (layered)
- UI design (invoke `/ui-ux-pro-max`)
- Operations architecture

**Output**:
- `docs/TECH_DESIGN.md` - Architecture design document
- `docs/TASK_LIST.md` - **REQUIRED** task breakdown with PRD closure mapping

**IMPORTANT**: TASK_LIST.md MUST be generated alongside TECH_DESIGN.md. Without tasks, development cannot proceed.

## Phase 3: Architecture Review (!pm-review)

**Role**: [Senior-Arch]

**MANDATORY**: `Skill: senior-architect`

**Purpose**: Multi-dimensional review. Identify minimum 3 issues or risks.

**Decision**:
- **Rejected**: Return to Phase 2 for revision
- **Approved**: Proceed to Phase 4

**Output**: Approved/Rejected decision

## Phase 4: Development (!pm-dev)

**Role**: [Dev]

**MANDATORY**: `Skill` tool:
- Backend task → `Skill: backend-developer`
- Frontend task → `Skill: frontend-developer`

### Task Execution Order

**Prerequisite Check (MANDATORY)**:
```
1. Check if docs/TASK_LIST.md exists
   └→ If NOT exists: BLOCKED - Return to Phase 2 (!pm-design) first
   └→ If exists but empty: BLOCKED - TASK_LIST must have tasks
   └→ If exists with tasks: Proceed
```

**Backend Tasks First:**
```
0. Environment Setup (FIRST!)
   └→ Create docker-compose-dev.yml with middleware services
   └→ Start middleware services (DB, Redis, etc.)
   └→ Verify services are running
   └→ Create .env from .env.example

1. Backend infrastructure (DB schema, config, migrations)
2. Backend APIs (REST endpoints, auth, business logic)
3. Backend unit tests
→ Backend must be deployable and runnable independently
```

**Important**: Environment setup MUST complete before any backend code to ensure tests can run without environment failures.

**Frontend Tasks Second:**
```
4. Frontend project setup (can start in parallel with backend)
5. Frontend UI components (blocked by corresponding backend API)
6. Frontend API integration & 联调 (blocked by frontend component AND backend API)
7. Frontend unit tests
```

### Frontend API Integration Task (IMPORTANT)

This is a **critical task type** that connects frontend to backend:

**Definition**: Task that binds frontend UI components to backend API endpoints

**Dependencies**:
```
Frontend API Integration Task blocked by:
├── Corresponding frontend UI component (must exist)
└── Corresponding backend API endpoint (must be stable)
```

**Example**:
```
| T-004 | User Login UI | frontend | Done | T-002 | @dev |
| T-009 | API Integration (Login) | frontend | Done | T-004, T-002 | @dev |
                         ↑                    ↑       ↑
                  Frontend Login UI    Backend Login API
```

**API Integration Steps**:
1. Verify backend API is stable and tested
2. Create API client function (e.g., `api/login.ts`)
3. Bind API client to UI component
4. Handle loading, error, success states
5. Write integration test
6. Verify end-to-end flow works

**Task Execution**:
1. Read `docs/TASK_LIST.md`
2. Pick backend tasks first (no blockers)
3. Execute TDD workflow for backend
4. When backend complete, switch to frontend tasks
5. For each frontend feature:
   - Implement UI component first
   - Then API integration task
6. Update task status
7. Notify [QA] when all tasks complete

### Key Principles

- Backend tasks complete before frontend API integration starts
- Frontend API integration blocked by both frontend component AND backend API
- Backend must be testable and runnable independently
- Each API integration task is a separate TDD cycle

**Output**: Code implemented, `docs/TASK_LIST.md` updated

## Phase 5: Testing (!pm-test)

**Role**: [QA]

**MANDATORY**: `Skill: qa-engineer`

**Testing Sequence** (defined in qa-engineer):
1. Backend tests first
2. Frontend tests second
3. API integration tests last

**Bug Handling**:
- Backend bug → Fix backend first
- Frontend bug → Fix frontend
- Re-test after fix until all pass

**Output**: Test report, `docs/ISSUE_LOG.md` updated

## Phase 6: Deployment (!pm-deploy)

**Role**: [DevOps]

**MANDATORY**: `Skill: devops-engineer`

**Deployment Order**:
1. Backend to test
2. Frontend to test
3. Staging
4. Production (after UAT)

**Rollback**: If health check fails → automatic rollback

**Output**: Deployed environments, `docs/DEPLOY_LOG.md` entry

## Phase 7: UAT (!pm-uat)

**Role**: [PM]

**MANDATORY**: `Skill: product-manager`

**Verify**: All P0 acceptance criteria from PRD

**Decision**:
- **Pass**: Task → `Done`
- **Fail**: Create issue → return to Phase 4 or Phase 1

**Output**: Task marked Done or issues created

## Hotfix Flow (!pm-hotfix)

**Role**: [Dev]

**MANDATORY**: `Skill: backend-developer` or `Skill: frontend-developer`

**Purpose**: Production bugs requiring expedited fix

**Flow**: Hotfix branch → Write test → Fix → Deploy to prod → Monitor

**Note**: Same quality standards as normal development

## Iteration Support

For large projects, decompose into iterations:

```
Iteration 1: MVP
  └─ Phase 1 → Phase 2 → Phase 3 → Phase 4 → Phase 5 → Phase 6 → Phase 7

Iteration 2: Enhanced Features
  └─ (Same flow)
```

Each iteration produces a working increment.

## State Files

| File | Status Flow | Managed By |
|:---|:---|:---|
| `docs/PRD.md` | Draft → Confirmed | [PM] |
| `docs/TECH_DESIGN.md` | Draft → Ready for Review → Rejected/Approved | [Arch] → [Senior-Arch] |
| `docs/TASK_LIST.md` | Pending → In-Progress → Blocked → To-Test → To-Deploy → Done | [Dev] |
| `docs/ISSUE_LOG.md` | Open → Fixed → Verified | [QA] |
| `docs/DEPLOY_LOG.md` | Deployment records | [DevOps] |

## Key Principles

1. **Skill Invocation is MANDATORY**: Always use `Skill` tool to invoke role skills
2. **Sequential phases**: Each phase must complete before next (except hotfix)
3. **Feedback loops**: Test failures → fix → re-test
4. **Task dependencies**: Use `blockedBy` to manage parallel work
5. **Backend first**: Backend complete before frontend integration
6. **Iteration-based**: Large projects decompose into iterations
7. **Quality gates**: Review and coverage thresholds must pass

## Skill References

Detailed implementation standards are defined in specialized skills:

| Phase | Skill | Contains |
|:---|:---|:---|
| Requirements | `product-manager` | Hidden requirements, implicit constraints |
| Technical Design | `solution-architect` | Task splitting, architecture design |
| Architecture Review | `senior-architect` | Review checklist, risk identification |
| Backend Development | `backend-developer` | Layered architecture, error handling, logging, security, unit tests, environment setup |
| Frontend Development | `frontend-developer` | Code org, component design, error handling, unit tests, visual/interaction standards |
| UI Design | `/ui-ux-pro-max` | Design system, typography, color, spacing |
| Frontend Implementation | `/frontend-design` | UI implementation, component specs |
| QA Testing | `qa-engineer` | Test coverage requirements, GStack integration, testing sequence |
| DevOps | `devops-engineer` | Deployment, monitoring, rollback |

## Example Usage

### Full-Stack Project
```
User: "Build an order management system"
Agent: !pm-analyze → Skill: product-manager → PRD created
Agent: !pm-design → Skill: solution-architect → TECH_DESIGN + TASK_LIST created
       └→ UI design: Skill: /ui-ux-pro-max
Agent: !pm-review → Skill: senior-architect → Approved

Agent: !pm-dev
       └→ Skill: backend-developer → Backend tasks complete
       └→ Skill: frontend-developer → Frontend tasks complete
       └→ Skill: /frontend-design → UI implemented

Agent: !pm-test → Skill: qa-engineer → Tests pass
       └─ If bugs: !pm-fix → Dev fixes → back to !pm-test

Agent: !pm-deploy → Skill: devops-engineer → Deployed
Agent: !pm-uat → Skill: product-manager → Accepted
```

### With Bug Found
```
!pm-test → BUG-001 found
  └→ Log ISSUE_LOG.md (Open)
  └→ !pm-fix → Dev fixes
  └→ Re-test → Pass
  └→ ISSUE_LOG.md → Fixed → Verified
  └→ Continue testing → All pass
```
