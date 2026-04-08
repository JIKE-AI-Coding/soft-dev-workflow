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

## Usage

This is a Claude Code skill. It is invoked automatically when you use trigger commands like `!pm-analyze`, `!pm-design`, etc.

For full usage examples, see `SKILL.md`.
