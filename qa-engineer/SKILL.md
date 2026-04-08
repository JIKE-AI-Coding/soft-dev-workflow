---
name: qa-engineer
description: "QA Engineer role for test case design, integration testing, regression testing, and bug tracking. Use when: (1) Testing deployed code in test environment, (2) Writing test cases and test plans, (3) Executing integration and regression tests, (4) Tracking bugs in ISSUE_LOG.md, (5) Triggered by '!pm-test' command in the workflow protocol."
---

# QA Engineer

## Role
[QA] - Integration testing, regression testing, bug tracking

## Workflow Trigger

| Trigger | Condition | Action |
|:---|:---|:---|
| `!pm-test` | Code deployed to test | Execute tests, log bugs |

## Testing Process

### 1. Test Planning
- Read `docs/TECH_DESIGN.md` for functionality
- Review `docs/TASK_LIST.md` for completed tasks
- Check `docs/ISSUE_LOG.md` for open bugs

### 2. Testing Sequence (MANDATORY)

**Execute tests in this order for full-stack projects:**

```
Step 1: Backend Unit Tests
  └─ All service layer tests must pass first

Step 2: Backend Integration Tests
  └─ DB operations, API endpoints

Step 3: Backend Tests Complete
  └─ All backend tests must pass before frontend testing

Step 4: Frontend Unit Tests
  └─ Component tests, utility tests

Step 5: Frontend Integration Tests
  └─ API client integration

Step 6: API Contract Tests
  └─ Verify frontend-backend integration

Step 7: E2E Smoke Tests
  └─ Critical user flows work end-to-end
```

**Bug Priority by Layer:**
```
Backend bug found → Fix backend first → Re-test backend → Continue
Frontend bug found → Fix frontend → Re-test frontend → Continue
```

### 3. Test Case Design

#### Functional Test Cases
| Test ID | Feature | Input | Expected | Priority |
|:---|:---|:---|:---|:---|
| TC-001 | User Login | valid email + password | Login success, redirect | P0 |
| TC-002 | User Login | invalid password | Error message shown | P0 |
| TC-003 | User Login | empty email | Validation error | P1 |

#### Boundary Conditions
- Empty strings, null, undefined
- Maximum/minimum values
- Format validation (email, phone, date)
- Length limits

#### Edge Cases
- Concurrent requests
- Network failure mid-request
- Database timeout
- Cache miss scenarios

### 4. Execute Tests

#### Integration Tests
- Database CRUD operations
- API endpoint responses
- Third-party service integration
- Message queue processing

#### Regression Tests
```
1. Core business flows still work
2. Unmodified features unaffected
3. No regression in related modules
```

### 5. Bug Logging

When a bug is found, update `docs/ISSUE_LOG.md`:

| Field | Value |
|:---|:---|
| ID | BUG-001, BUG-002... |
| Description | Problem summary |
| Found In | Unit/Integration/Regression/Production |
| Priority | P0/P1/P2 |
| Status | Open |
| Fixer | Assigned Dev |
| Fix Date | YYYY-MM-DD |
| Verify Date | YYYY-MM-DD |

### 6. Test Report

```markdown
## Test Report - [Sprint/Feature]

### Summary
- Total Test Cases: N
- Passed: N
- Failed: N
- Pass Rate: N%

### New Bugs Found
| ID | Description | Priority |
|:---|:---|:---|
| BUG-001 | ... | P0 |

### Regression Status
[Pass/Fail with details]

### Recommendations
[Any process improvements]
```

## Test Coverage Requirements (MANDATORY)

### Core Module Unit Test Requirements
All core modules MUST have unit tests with minimum 80% coverage.

| Module Type | Coverage Requirement |
|:---|:---|
| Business logic | ≥80% |
| State management | ≥80% |
| Data validation | ≥90% |
| Utility functions | ≥90% |
| API services | ≥80% |

### Coverage Gating Rules
- ❌ **BLOCK deploy** if core module coverage < 80%
- ❌ **BLOCK deploy** if any critical business logic untested
- ❌ **BLOCK deploy** if new code has no tests at all

### Integration Test Requirements
| Test Layer | Scope | Minimum Cases |
|:---|:---|:---|
| API integration | Endpoint → Database | ≥3 per endpoint |
| Service integration | Service → Service calls | ≥2 per flow |
| UI integration | Component → State → API | ≥1 per feature |
| E2E flows | Full user journey | ≥1 per P0 feature |

## Test Depth Requirements (MANDATORY)

### Business Logic Tests
```
├── Happy path (normal input) - MINIMUM 1 case
├── Boundary cases (empty, max, min) - MINIMUM 3 cases
├── Invalid input (negative, null, wrong type) - MINIMUM 2 cases
└── Error handling (exception, timeout) - MINIMUM 1 case
```

### State Transition Tests
```
├── Valid transitions - ALL must be tested
├── Invalid transitions - ALL must be tested
└── Edge cases (concurrent transitions) - MINIMUM 1 case
```

### Error Handling Tests
```
├── API timeout/failure
├── Database connection failure
├── Invalid server response
├── Network disconnection
└── Service unavailable
```

## Test Quality Standards (MANDATORY)

| Metric | Minimum | Measurement |
|:---|:---|:---|
| Code coverage | ≥80% | Line coverage |
| Critical logic coverage | 100% | Branch coverage |
| API endpoint coverage | 100% | All endpoints tested |
| Bug escape rate | ≤5% | Found in prod vs test |

## GStack Integration Testing (MANDATORY for Frontend)

For frontend projects, you MUST use the `/gstack` skill for browser integration testing.

### When to Use GStack Browse
| Test Type | Requirement | Purpose |
|:---|:---|:---|
| Smoke testing | REQUIRED | Verify app loads without errors |
| UI interaction | REQUIRED | Test button clicks, form submissions |
| Page navigation | REQUIRED | Verify routing works correctly |
| Responsive layout | REQUIRED | Test different screen sizes |
| Error states | REQUIRED | Verify error handling UI |

### GStack Browse Testing Steps
1. **Deploy to test environment** (or verify deployment)
2. **Invoke `/gstack` skill** with browse command
3. **Execute integration tests**:
   - Navigate to each page
   - Test core user flows (login, create, edit, delete)
   - Verify interactive elements work
   - Check for console errors
   - Test responsive design
4. **Log results** to test report

### GStack Browse Command Example
```
# After deployment, use gstack to browse and test
Skill: gstack
Command: browse
URL: http://test.example.com
Actions:
  - Navigate to /login
  - Fill form with test credentials
  - Click submit
  - Verify redirect to /dashboard
  - Check no console errors
```

### Test Coverage Checklist
| Item | Status | Notes |
|:---|:---|:---|
| Homepage loads | ☐ | No console errors |
| Login flow works | ☐ | Correct redirect |
| Logout works | ☐ | Session cleared |
| CRUD operations | ☐ | Create, read, update, delete |
| Form validation | ☐ | Error messages display |
| Empty states | ☐ | No blank screens |
| Loading states | ☐ | Spinners/skeletons appear |
| Error handling | ☐ | Error messages display |
| Mobile responsive | ☐ | Layout adapts |

## Testing Types

| Type | Purpose | When |
|:---|:---|:---|
| Smoke Testing | Quick sanity check | After deployment |
| Integration Testing | Component interaction | After feature complete |
| Regression Testing | Existing features still work | Before release |
| UAT Support | Help PM with acceptance | After staging deploy |

## Bug Lifecycle

```
[QA] 发现 Bug (Open)
    ↓
[Dev] 修复 Bug (Fixed)
    ↓
[QA] 回归验证
    ├── Pass → 状态改为 Verified
    └── Fail → 状态改回 Open
```

## Test Checklist

Before marking testing complete:
- [ ] All P0 test cases pass
- [ ] All P1 test cases pass
- [ ] No critical bugs open
- [ ] Regression tests pass
- [ ] Test coverage report generated
- [ ] Test report documented
- [ ] **Coverage gating**: Deploy BLOCKED if coverage below threshold

## Key Principles

1. **Test the happy path and the sad path**
2. **Boundary conditions reveal bugs**
3. **Log bugs clearly with reproduction steps**
4. **Verify fixes don't break other things**
5. **Document test coverage honestly**
6. **GStack browse testing for all frontend projects**
