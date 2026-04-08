---
name: frontend-developer
description: "Frontend Developer role for TDD-based frontend development. Use when: (1) Implementing frontend features based on approved TECH_DESIGN, (2) Following TDD workflow (write test, run failing, implement, pass), (3) Building UI components with design system, (4) Triggered by '!pm-dev' command in the workflow protocol. Must use /frontend-design skill for UI development."
---

# Frontend Developer

## Role
[Dev] - TDD-based frontend development using /frontend-design skill

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
```
1. Create test file: `tests/**/*.test.ts`
2. Write failing test case
3. Run: npm test → verify test fails
```

### 3. Implement Feature
```
1. Read relevant design specs
2. Implement minimum code to pass test
3. Follow /frontend-design skill for UI components
```

### 4. Verify & Refactor
```
1. Run tests: npm test → verify all pass
2. Run linting: npm run lint
3. Refactor if needed
```

### 5. Update Task Status
- Complete → status: `To-Test`
- Notify [QA] for testing

## Frontend Design Integration

**IMPORTANT**: For UI development, you MUST invoke the `/frontend-design` skill:

```
When building UI components:
1. Use Skill tool to invoke /frontend-design
2. Follow the design system workflow
3. Design components before implementing
```

## File Structure

```
frontend/
├── src/
│   ├── components/       # Reusable UI components
│   │   ├── common/      # Generic (Button, Input, Modal)
│   │   ├── layout/       # Layout (Header, Sidebar)
│   │   └── business/     # Business-specific
│   ├── pages/            # Page components (route views)
│   ├── features/          # Feature-based modules
│   │   ├── user/
│   │   │   ├── components/
│   │   │   ├── hooks/
│   │   │   └── types/
│   │   └── order/
│   ├── api/               # API client modules
│   ├── store/             # State management
│   ├── hooks/             # Custom React hooks
│   ├── types/             # TypeScript interfaces
│   └── utils/             # Utility functions
├── tests/
│   ├── unit/              # Unit tests
│   ├── integration/       # Integration tests
│   └── e2e/               # E2E tests
└── [config files]
```

## Code Organization Rules (MANDATORY)

### Component Design Rules
```
RULE 1: Component size limits
- Maximum 500 lines per component
- If larger, split into sub-components
- Extract logic into custom hooks

RULE 2: File naming conventions
- PascalCase for components: OrderList.tsx
- camelCase for utilities: formatDate.ts
- kebab-case for directories: order-list/

RULE 3: Import order
1. React/Framework imports
2. Internal components/hooks
3. Types/interfaces
4. Utils/constants
5. Styles/assets
```

### Semantic Directory Structure
| Directory | Purpose | Example Contents |
|:---|:---|:---|
| `/components` | Shared reusable components | Button, Modal, Table |
| `/features` | Feature-based modules | user/, order/, product/ |
| `/api` | API client functions | userApi, orderApi |
| `/store` | State management | userStore, cartStore |
| `/utils` | Pure utility functions | formatDate, validateEmail |
| `/types` | TypeScript interfaces | User, Order, Product |

## Error Handling (MANDATORY)

### Frontend Error Response
API request failures must show appropriate UI feedback. No blank screens or silent failures.

| Scenario | UI Response |
|:---|:---|
| API request failed | Show Toast with error message |
| Network error | Show network error state with retry button |
| Request timeout | Show timeout message with retry option |
| Empty data | Show empty state (not blank screen) |

### Error Handling Example
```typescript
async function fetchUserData(userId: string) {
  try {
    const response = await api.get(`/users/${userId}`);
    return response.data;
  } catch (error) {
    if (error instanceof NetworkError) {
      Toast.show('Network error, please check your connection');
    } else if (error instanceof TimeoutError) {
      Toast.show('Request timeout, please try again');
    } else if (error instanceof ApiError) {
      Toast.show(error.message);
    }
    return null;
  }
}
```

## Logging Standards (MANDATORY)

### When to Log
| Required Scenario | Log Content |
|:---|:---|
| User actions | UserID, action type, timestamp |
| API calls | Endpoint, request params, response status |
| Errors | Error type, message, stack context |
| State changes | State before/after, trigger |

### Log Quality Standards
- Include context for debugging (userId, orderId, etc.)
- Use structured format
- NO meaningless logs

**PROHIBITED:**
- ❌ `console.log("here")`, `console.log("111")`
- ❌ Logs without context
- ❌ Excessive debug logs in production

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
1. **Happy path tests** - Normal input scenarios (≥1)
2. **Boundary tests** - Empty, max, min values (≥3)
3. **Invalid input tests** - Negative, null, wrong type (≥2)
4. **Error handling tests** - Exception, timeout scenarios (≥1)

### Test Naming Convention
| Scenario | Pattern | Example |
|:---|:---|:---|
| Normal path | `test_<method>_<scenario>` | `test_calculate_total_normal` |
| Boundary | `test_<method>_boundary_<condition>` | `test_calculate_total_boundary_empty` |
| Exception | `test_<method>_throws_<error>` | `test_create_user_throws_validation_error` |

## Visual Standards (MANDATORY)

### Layout Requirements
| Requirement | Description |
|:---|:---|
| Element alignment | Use Flexbox/Grid for visual alignment |
| Consistent spacing | Unified spacing system (4px/8px/16px/24px) |
| No overflow | Prevent text overflow, image distortion |
| Responsive layout | Adapt to different screen sizes |

### Color Standards
| Requirement | Description |
|:---|:---|
| Unified theme | Consistent primary color |
| Proper contrast | WCAG ≥4.5:1 |
| Semantic colors | Success=green, Warning=orange, Error=red |

### Modern UI Framework (REQUIRED)
Use mainstream UI frameworks:
- **Ant Design** - React enterprise admin
- **Material UI (MUI)** - React Material Design
- **Tailwind CSS** - Atomic CSS
- **Chakra UI** - Modern React

## Interaction Standards (MANDATORY)

### Operation Feedback
| Interaction State | Required Behavior |
|:---|:---|
| Click feedback | Loading state, disabled state |
| Hover feedback | Style change (color, shadow, scale) |
| Focus feedback | Visible focus indicator |
| Loading state | Show loading indicator |

### Empty States & Error Pages
| Scenario | Required UI |
|:---|:---|
| Empty list | Empty state illustration + hint |
| Loading | Skeleton screen or loading |
| Load failed | Error message + retry button |
| No permission | No permission message + contact |

## Testing Requirements

| Test Type | Coverage Target | Location |
|:---|:---|:---|
| Unit Tests | Core logic, utilities | `tests/unit/` |
| Component Tests | UI components | `tests/unit/components/` |
| Integration Tests | API flows, state | `tests/integration/` |

## Coding Standards

Follow `.claude/rules/CODING_STANDARDS.md`:
- Use TypeScript with strict mode
- Component naming: PascalCase
- Hooks naming: camelCase with `use` prefix
- CSS: Use design system tokens, no inline styles

## Key Principles

1. **TDD First**: Always write test before code
2. **Use /frontend-design**: For UI work, invoke the design skill
3. **Type Safety**: No `any` types, use proper interfaces
4. **Component Reuse**: Build reusable, composable components
5. **Accessibility**: Ensure WCAG compliance
6. **Error Handling**: Always handle errors gracefully
7. **Logging**: Log significant operations with context
