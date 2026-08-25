---
work_package_id: WP15
title: Customer Portal & Authentication UI
dependencies:
- WP02
- WP04
- WP14
requirement_refs:
- FR-001
- FR-013
- FR-020
- FR-045
tracker_refs: []
planning_base_branch: epic1
merge_target_branch: epic1
branch_strategy: Planning artifacts for this mission were generated on epic1. During /spec-kitty.implement this WP may branch from a dependency-specific base, but completed changes must merge back into epic1 unless the human explicitly redirects the landing branch.
subtasks:
- T060
- T061
agent: claude
history:
- timestamp: '2026-08-25T18:45:19Z'
  action: created
  actor: spec-kitty-tasks
agent_profile: implementer-ivan
authoritative_surface: frontend/src/pages/
create_intent:
- frontend/src/pages/LoginPage.tsx
- frontend/src/pages/CustomerDashboard.tsx
- frontend/src/pages/CustomerCaseList.tsx
- frontend/src/pages/CustomerCaseDetail.tsx
- frontend/src/pages/CreateCasePage.tsx
- frontend/src/contexts/AuthContext.tsx
- frontend/src/hooks/useAuth.ts
- frontend/src/services/auth.api.ts
- frontend/src/services/cases.api.ts
execution_mode: code_change
model: claude-sonnet-4-6
owned_files:
- frontend/src/pages/LoginPage.tsx
- frontend/src/pages/CustomerDashboard.tsx
- frontend/src/pages/CustomerCaseList.tsx
- frontend/src/pages/CustomerCaseDetail.tsx
- frontend/src/pages/CreateCasePage.tsx
- frontend/src/contexts/AuthContext.tsx
- frontend/src/hooks/useAuth.ts
- frontend/src/services/auth.api.ts
- frontend/src/services/cases.api.ts
role: implementer
tags: []
---

## ⚡ Do This First: Load Agent Profile

```bash
/opencode/skill ad-hoc-profile-load --name implementer-ivan
```

---

## Objective

Build customer-facing portal: authentication, case list, case detail, create case, and communications.

---

## Context

- **Dependencies**: WP02 (auth endpoints), WP04 (case endpoints), WP14 (frontend setup)
- **Reference Documents**: spec.md (FR-001, FR-013, FR-020, FR-045), contracts/openapi.yaml (Auth, Cases, Communications tags)

---

## Detailed Guidance per Subtask

### T060: Create AuthContext, useAuth hook, login page, protected routes

**Purpose**: Authentication UI per FR-001.

**Steps**:
1. Create `src/contexts/AuthContext.tsx`:
   - State: user, accessToken, refreshToken, isLoading
   - Actions: login, logout, refresh, updateUser
   - Persist tokens in localStorage (access) + httpOnly cookie (refresh) or both in memory + localStorage
   - Auto-refresh on token expiry (interceptor)
2. Create `src/hooks/useAuth.ts` - hook to access context
4. Create `src/services/auth.api.ts` - axios instance with interceptors:
   - Attach access token
   - On 401: try refresh, retry once
5. Create `src/pages/LoginPage.tsx`:
   - Form with email, password, rememberMe
   - Validation with Zod
   - Redirect to customer dashboard on success
6. Protected route wrapper: `RequireAuth` + `RequireRole`

**Files**: `frontend/src/contexts/AuthContext.tsx`, `frontend/src/hooks/useAuth.ts`, `frontend/src/services/auth.api.ts`, `frontend/src/pages/LoginPage.tsx`, `frontend/src/router/ProtectedRoute.tsx`

**Validation**:
- [ ] Login form works
- [ ] Tokens stored and persisted
- [ ] Auto-refresh on expiry
- [ ] Protected routes redirect to login
- [ ] Role-based route access

---

### T061: Build customer portal: case list, case detail, create case, communications

**Purpose**: Customer case management per FR-013, FR-020, FR-045.

**Steps**:
1. Create `src/pages/CustomerDashboard.tsx`:
   - Case list: my cases with status, subject, priority, updated_at
   - Filter: status, date range
   - Pagination
   - Real-time updates (WP18): new cases, status changes
2. Create `src/pages/CreateCasePage.tsx`:
   - Form: subject, description, category (optional), attachment upload
   - Submit → POST /cases
   - Success → redirect to case detail
3. Create `src/pages/CustomerCaseDetail.tsx`:
   - Case header: number, subject, status, priority, created_at
   - Communications thread: chronological, customer-visible only
   - Reply form: textarea, attachment upload, submit
   - Real-time: new communications appear
4. Create `src/services/cases.api.ts` - typed API client for cases
5. Create `src/services/communications.api.ts` - for communications

**Files**: `frontend/src/pages/CustomerDashboard.tsx`, `frontend/src/pages/CreateCasePage.tsx`, `frontend/src/pages/CustomerCaseDetail.tsx`, `frontend/src/services/cases.api.ts`, `frontend/src/services/communications.api.ts`

**Validation**:
- [ ] Customer sees only own cases
- [ ] Case list filters and paginates
- [ ] Create case with attachment works
- [ ] Case detail shows communications chronologically
- [ ] Customer can reply to case
- [ ] Real-time updates (SSE) work

---

## Branch Strategy

- **Planning Branch**: `epic1`
- **Work Package Branch**: `feature/wp15-customer-portal` (from `epic1`)
- **Final Merge Target**: `epic1`
- **Implementation Command**: `spec-kitty agent action implement WP15 --agent claude`

---

## Test Strategy

- Unit tests for auth context/hooks
- Component tests for pages
- Integration test: login → create case → view → reply
- E2E test (WP19)

---

## Definition of Done

- [ ] Login page with validation
- [ ] Auth context with token management
- [ ] Customer dashboard with case list
- [ ] Create case with attachments
- [ ] Case detail with communications
- [ ] Customer sees only own data
- [ ] Real-time updates
- [ ] Tests pass

---

## Risks

- Auth state persistence across refresh
- Protected route guards
- Case creation UX
- Communication thread display
- Token refresh race conditions

---

## Reviewer Guidance

- Verify customer cannot access other customers' cases
- Check token refresh logic
- Confirm attachment upload works
- Test real-time communication updates