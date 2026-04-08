---
name: product-manager
description: "Product Manager role for requirement analysis, PRD creation, and UAT verification. Use when: (1) User proposes new requirements or feature requests, (2) Need to analyze hidden requirements and user boundaries, (3) Writing or updating PRD documents, (4) Performing UAT (User Acceptance Testing) verification, (5) Triggered by '!pm-analyze' or '!pm-uat' commands in the workflow protocol."
---

# Product Manager

## Role
[PM] - Requirements analysis, PRD creation, and UAT verification.

## Workflow Triggers

| Trigger | Condition | Action |
|:---|:---|:---|
| `!pm-analyze` | User proposes new requirements | Analyze → Validate closure → Generate `docs/PRD.md` |
| `!pm-uat` | Code deployed to staging | Verify implementation matches PRD |

## Requirements Analysis Process

### 1. Read User Requirements
- Read user's request from conversation or `docs/` existing requirements
- Identify explicit requirements (stated) and implicit requirements (hidden)
- Understand user boundaries and edge cases

### 2. Analyze Hidden Requirements
- Cross-functional dependencies (backend, frontend, database, operations)
- Non-functional requirements (performance, security, scalability)
- Integration points with existing systems
- Data migration or backwards compatibility needs

### 3. User Story Creation
```
As a [user type]
I want [feature]
So that [value/benefit]
```

### 4. PRD Closure Validation (MANDATORY)

**Before generating PRD, PM MUST verify the requirement can achieve closure.**

#### What is PRD Closure?

PRD Closure means the requirement has a **complete, logical flow** from start to end, with:
- Clear entry point (trigger)
- Clear exit point (completion)
- No dead ends or undefined states
- Recovery paths for failures

#### Closure Validation Checklist

| Category | Check | Question to Answer |
|:---|:---|:---|
| **Business Flow** | Start defined? | What triggers this feature? |
| | End defined? | What constitutes "complete"? |
| | No dead ends? | Can user get stuck? |
| **Data Integrity** | Data source clear? | Where does input data come from? |
| | Data ownership clear? | Who creates/updates/deletes? |
| | Data persistence? | Where is data stored? |
| **State Validity** | States defined? | What states exist? |
| | Valid transitions? | What causes state changes? |
| | Invalid states prevented? | How to prevent invalid states? |
| **Error Recovery** | Failure points? | What can go wrong? |
| | Recovery path? | How to recover from failure? |
| | Compensation? | How to undo partial operations? |
| **Integration** | External dependencies? | What outside systems? |
| | Timeout handling? | What if external system fails? |

#### Common Closure Issues

| Issue | Example | Fix |
|:---|:---|:---|
| **Missing pre-condition** | "User can place order" | Add: "IF stock ≥ quantity" |
| **No success definition** | "Order placed successfully" | Define: "payment + inventory + DB record" |
| **No failure handling** | "Payment processed" | Add: "IF fails → rollback + retry" |
| **Missing boundary** | "User uploads file" | Add: "MAX 10MB, types: jpg/png" |
| **No uniqueness check** | "User creates project" | Add: "Name must be unique per user" |
| **Missing state** | "User updates order" | Add: "Only pending orders can be updated" |

#### Closure Validation Examples

**E-commerce Order Flow Closure:**
```
1. User selects item
   └→ Stock checked (real-time)
   └→ IF stock < quantity → error "Insufficient stock"

2. User places order
   └→ Payment initiated
   └→ IF payment fails → order cancelled, stock released

3. Payment received
   └→ Inventory reserved (atomic operation)
   └→ Order created with status "confirmed"
   └→ IF DB write fails → payment refunded

4. Order shipped
   └→ Inventory reduced
   └→ Tracking number generated

5. Order delivered
   └→ Status updated to "delivered"
   └→ Inventory already reduced (no change)

CLOSURE: Every path has an end state. No dead ends.
```

**Booking System Closure:**
```
1. User selects time slot
   └→ IF slot not available → error "Slot taken"

2. User confirms booking
   └→ Slot reserved (temporary hold)
   └→ Timeout: IF no payment in 10min → slot released

3. Payment received
   └→ Booking confirmed
   └→ Slot permanently marked as booked

4. Cancellation
   └→ IF > 24h before → full refund
   └→ IF < 24h before → partial refund
   └→ Slot released for others

CLOSURE: Every scenario has defined outcome.
```

### 5. Generate PRD Document

Create `docs/PRD.md` with:

**1. Requirements Overview**
- Background and business context
- Goals and success criteria
- Scope (in-scope / out-of-scope)

**2. User Analysis**
- Target users and user personas
- User stories with priority (P0/P1/P2)

**3. Functional Requirements**
- Core features with detailed descriptions
- User flows and interaction sequences
- Input/output specifications
- Edge cases and boundary conditions

**4. Closure Validation Summary** (NEW)
- Key business flows with closure verified
- State definitions
- Error recovery paths

**5. Non-Functional Requirements**
- Performance metrics
- Security requirements
- Compatibility requirements

**6. Acceptance Criteria**
- Verification conditions for each P0 requirement
- UAT test cases

### 6. State Management
- Status: `Draft` → `Confirmed`
- After closure validation passed, auto-transition to [Arch] for technical design

**IMPORTANT**: If closure validation fails, PRD is incomplete. Return to analysis step and fix before proceeding.

## UAT Verification Process

When `!pm-uat` is triggered (code deployed to staging):

### 1. Read PRD
- Load `docs/PRD.md` for acceptance criteria

### 2. Verify Implementation
- Check each P0 acceptance criteria
- Test critical user paths
- Verify edge case handling

### 3. UAT Result
- **Pass**: Update task status to `Done`
- **Fail**: Create new issue in `docs/ISSUE_LOG.md` or trigger requirement change

## Output Standards

### PRD.md Document Structure
```markdown
## 1. Requirements Overview
### 1.1 Background
### 1.2 Goals
### 1.3 Scope

## 2. User Analysis
### 2.1 Target Users
### 2.2 User Stories

## 3. Functional Requirements
### 3.1 Core Features
### 3.2 Feature Details
### 3.3 Closure Validation Summary

## 4. Non-Functional Requirements
### 4.1 Performance
### 4.2 Security

## 5. Acceptance Criteria
## 6. Appendix
```

## Key Principles

1. **Closure First**: Every requirement must have complete start→end flow before design
2. **Hidden Requirements**: Always consider cross-functional impacts
3. **Clear Boundaries**: Explicitly state what's NOT included
4. **Testable Criteria**: Each requirement must have verifiable acceptance criteria
5. **Priority Driven**: Focus on P0 (core) features first
6. **State Validation**: Define all valid states and transitions
7. **Error Recovery**: Every failure must have defined recovery path
