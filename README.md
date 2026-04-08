# Development Workflow Orchestrator

> **中文文档**: [README_CN.md](README_CN.md)

A Claude Code skill that orchestrates the complete software development lifecycle from requirements analysis through deployment.

## Overview

This skill implements a 7-phase workflow pipeline with clearly defined roles:

```
[PM] → [Arch] → [Senior-Arch] → [Dev] → [QA] → [DevOps] → [PM]
  ↓        ↓            ↓              ↓         ↓          ↓
!pm-analyze !pm-design !pm-review    !pm-dev   !pm-test  !pm-deploy
                                                                         !pm-uat
```

## Trigger Commands

| Command | Role | Purpose |
|:---|:---|:---|
| `!pm-analyze` | [PM] | Requirements analysis, generate PRD |
| `!pm-design` | [Arch] | Technical design, generate TECH_DESIGN.md + TASK_LIST.md |
| `!pm-review` | [Senior-Arch] | Architecture review and approval |
| `!pm-dev` | [Dev] | TDD development (backend first, then frontend) |
| `!pm-test` | [QA] | Integration testing, bug tracking |
| `!pm-deploy` | [DevOps] | Deploy to test/staging/production |
| `!pm-uat` | [PM] | User acceptance testing |
| `!pm-hotfix` | [Dev] | Production bug expedited fix |

## Workflow Phases

### Phase 1: Requirements (!pm-analyze)
- Invokes: `product-manager` skill
- Outputs: `docs/PRD.md`
- Validates business closure, data integrity, state validity

### Phase 2: Technical Design (!pm-design)
- Invokes: `solution-architect` skill
- Outputs: `docs/TECH_DESIGN.md`, `docs/TASK_LIST.md`
- Includes UI design via `/ui-ux-pro-max`

### Phase 3: Architecture Review (!pm-review)
- Invokes: `senior-architect` skill
- Decision: Approved or Rejected (returns to Phase 2)

### Phase 4: Development (!pm-dev)
- Backend tasks via `backend-developer`
- Frontend tasks via `frontend-developer`
- Frontend UI via `/frontend-design`
- **Order**: Backend first → Frontend second
- **API Integration**: Blocked by both frontend component AND backend API

### Phase 5: Testing (!pm-test)
- Invokes: `qa-engineer` skill
- Outputs: `docs/ISSUE_LOG.md`
- Bug lifecycle: Open → Fixed → Verified

### Phase 6: Deployment (!pm-deploy)
- Invokes: `devops-engineer` skill
- Order: Test → Staging → Production
- Rollback on health check failure

### Phase 7: UAT (!pm-uat)
- Invokes: `product-manager` skill
- Verifies P0 acceptance criteria from PRD

## State Files

| File | Status Flow | Managed By |
|:---|:---|:---|
| `docs/PRD.md` | Draft → Confirmed | [PM] |
| `docs/TECH_DESIGN.md` | Draft → Ready for Review → Rejected/Approved | [Arch] → [Senior-Arch] |
| `docs/TASK_LIST.md` | Pending → In-Progress → Blocked → To-Test → To-Deploy → Done | [Dev] |
| `docs/ISSUE_LOG.md` | Open → Fixed → Verified | [QA] |

## Key Principles

1. **Skill Invocation is MANDATORY**: Use `Skill` tool to delegate to role skills
2. **Sequential phases**: Each phase must complete before next
3. **Backend first**: Backend complete before frontend integration
4. **Iteration-based**: Large projects decompose into iterations
5. **PRD Closure Validation**: Every feature must have complete start→end flow

## External Dependencies

This skill depends on the following external Claude Code skills:

| Skill | Purpose | Used In Phase |
|:---|:---|:---|
| `ui-ux-pro-max` | UI/UX design system, component specifications, design review | Phase 2: Technical Design |
| `frontend-design` | Frontend development with design system, component implementation | Phase 4: Development |

## Usage

This is a Claude Code skill. It is invoked automatically when you use trigger commands like `!pm-analyze`, `!pm-design`, etc.

### How to Start

1. **Start with requirements**: Tell Claude about your project idea or feature request
2. **Use trigger commands**: Invoke workflow phases with `!pm-*` commands
3. **Follow the pipeline**: Each phase outputs artifacts used by the next

### Basic Workflow Example

```
1. User: "我需要一个用户登录功能"
2. User: "!pm-analyze"  → [PM] generates PRD
3. User: "!pm-design"   → [Arch] generates TECH_DESIGN + TASK_LIST
4. User: "!pm-review"   → [Senior-Arch] reviews and approves
5. User: "!pm-dev"      → [Dev] implements with TDD
6. User: "!pm-test"     → [QA] tests and reports bugs
7. User: "!pm-deploy"   → [DevOps] deploys
8. User: "!pm-uat"      → [PM] accepts
```

### Trigger Command Examples

#### Start Requirements Analysis
```
!pm-analyze
```
Claude will act as [PM] and generate `docs/PRD.md` from your requirements.

#### Start Technical Design
```
!pm-design
```
Claude will act as [Arch] and generate `docs/TECH_DESIGN.md` and `docs/TASK_LIST.md`.

#### Review Technical Design
```
!pm-review
```
Claude will act as [Senior-Arch] and review the technical design. If rejected, it returns to design phase.

#### Implement Development
```
!pm-dev
```
Claude will act as [Dev], reading TASK_LIST and implementing tasks with TDD (test → fail → code → pass).

#### Run Testing
```
!pm-test
```
Claude will act as [QA], run integration tests, and log bugs to `docs/ISSUE_LOG.md`.

#### Deploy to Environment
```
!pm-deploy
```
Claude will act as [DevOps], deploying to test → staging → production.

#### User Acceptance Testing
```
!pm-uat
```
Claude will act as [PM], verifying the implementation against PRD acceptance criteria.

### Typical Project Flow

```
┌─────────────────────────────────────────────────────────────┐
│  Phase 1: Requirements Analysis                              │
│  Input: User's project idea                                   │
│  Output: docs/PRD.md (Draft → Confirmed)                     │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  Phase 2: Technical Design                                   │
│  Input: Confirmed PRD                                        │
│  Output: docs/TECH_DESIGN.md, docs/TASK_LIST.md              │
│  External: ui-ux-pro-max skill for UI design                │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  Phase 3: Architecture Review                               │
│  Input: TECH_DESIGN.md                                      │
│  Output: Approved/Rejected                                  │
│  If Rejected → return to Phase 2                            │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  Phase 4: Development                                        │
│  Input: TASK_LIST.md                                        │
│  Output: Implemented code, tests passing                    │
│  Order: Backend first → Frontend second                    │
│  External: frontend-design skill for UI                    │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  Phase 5: Testing                                            │
│  Input: Deployed code                                       │
│  Output: Test report, docs/ISSUE_LOG.md                     │
│  Bug flow: Open → Fixed → Verified                          │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  Phase 6: Deployment                                         │
│  Order: Test → Staging → Production                         │
│  Rollback on health check failure                           │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  Phase 7: UAT                                                │
│  Input: Deployed to staging                                 │
│  Output: PM acceptance (Done or Rejected)                    │
└─────────────────────────────────────────────────────────────┘
```
