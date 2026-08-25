---
work_package_id: WP02
title: Authentication & Authorization Foundation (Epic 1)
dependencies:
- WP01
requirement_refs:
- C-009
- FR-001
- FR-002
- FR-003
- FR-004
- FR-005
- FR-006
tracker_refs: []
planning_base_branch: epic1
merge_target_branch: epic1
branch_strategy: Planning artifacts for this mission were generated on epic1. During /spec-kitty.implement this WP may branch from a dependency-specific base, but completed changes must merge back into epic1 unless the human explicitly redirects the landing branch.
subtasks:
- T006
- T007
- T008
- T009
- T010
- T011
agent: claude
history:
- timestamp: '2026-08-25T18:45:19Z'
  action: created
  actor: spec-kitty-tasks
agent_profile: implementer-ivan
authoritative_surface: backend/src/auth/
create_intent: []
execution_mode: code_change
model: claude-sonnet-4-6
owned_files:
- backend/src/auth/**
- backend/src/users/**
- backend/src/common/guards/**
- backend/src/common/decorators/**
- backend/src/common/strategies/**
role: implementer
tags: []
---

## ⚡ Do This First: Load Agent Profile

Before reading any other context, load your assigned agent profile:

```bash
/opencode/skill ad-hoc-profile-load --name implementer-ivan
```

This will apply the implementer persona with its identity, governance scope, boundaries, and initialization declarations. Do not proceed until the profile is loaded.

---

## Objective

Implement complete JWT authentication with access/refresh tokens using Argon2id password hashing, RBAC middleware for 4 roles (customer, agent, supervisor, admin), session management with refresh token rotation/revocation, and rate limiting on auth endpoints.

---

## Context

- **Mission**: customer-support-platform-mvp-01M0DPMQ
- **Phase**: Epic 1 - Authentication & Access (primary deliverable)
- **Dependencies**: WP01 (NestJS, Prisma, Docker Compose running)
- **Key Decisions**: JWT with refresh tokens (DM-01M0DQPC), RBAC roles per spec
- **Reference Documents**: spec.md (FR-001 through FR-006, C-009), plan.md (IC-01), data-model.md (User, UserRole, Session), contracts/openapi.yaml (Auth, Users tags), research.md (JWT refresh token rotation edge cases)

---

## Detailed Guidance per Subtask

### T006: Implement JWT authentication with access/refresh tokens (Argon2id)

**Purpose**: Core authentication mechanism per C-009 and DM-01M0DQPC.

**Steps**:
1. Create `src/auth/auth.module.ts` with JwtModule, PassportModule registration
2. Create `src/auth/strategies/jwt.strategy.ts` - JWT access token validation:
   - Extract from Authorization: Bearer header
   - Verify signature with secret from config
   - Validate expiration (15 min access token)
   - Attach user to request with roles
3. Create `src/auth/strategies/jwt-refresh.strategy.ts` - Refresh token validation:
   - Extract from request body
   - Verify against hashed refresh token in Session table
   - Check not revoked, not expired (7 days)
4. Create `src/auth/services/token.service.ts`:
   - `generateAccessToken(user: User, roles: string[]): string` - 15 min, HS256
   - `generateRefreshToken(): string` - cryptographically random
   - `hashRefreshToken(token: string): Promise<string>` - Argon2id
   - `verifyRefreshToken(plain: string, hash: string): Promise<boolean>`
   - `decodeToken(token: string): JwtPayload | null`
5. Create `src/auth/services/password.service.ts`:
   - `hashPassword(password: string): Promise<string>` - Argon2id (memory: 65536, iterations: 3, parallelism: 4)
   - `verifyPassword(password: string, hash: string): Promise<boolean>`
6. Create `src/auth/services/session.service.ts`:
   - `createSession(userId, refreshTokenHash, userAgent, ip): Promise<Session>`
   - `revokeSession(sessionId): Promise<void>`
   - `revokeAllUserSessions(userId): Promise<void>`
   - `cleanupExpiredSessions(): Promise<void>` (cron job)

**Files**: `backend/src/auth/strategies/`, `backend/src/auth/services/token.service.ts`, `backend/src/auth/services/password.service.ts`, `backend/src/auth/services/session.service.ts`, `backend/src/auth/auth.module.ts`

**Validation**:
- [ ] Access token expires in 15 minutes
- [ ] Refresh token expires in 7 days
- [ ] Argon2id hashing works (verify known hash)
- [ ] Session created on login, revoked on logout
- [ ] Expired sessions cleaned up

---

### T007: Build login, refresh, logout, me endpoints per OpenAPI contracts

**Purpose**: REST endpoints matching contracts/openapi.yaml Auth tag.

**Steps**:
1. Create `src/auth/dto/login.dto.ts` - Zod schema for LoginRequest
2. Create `src/auth/dto/refresh.dto.ts` - Zod schema for RefreshTokenRequest
3. Create `src/auth/controllers/auth.controller.ts`:
   - `POST /auth/login` - validate credentials, create session, return tokens + user
   - `POST /auth/refresh` - rotate refresh token, return new access+refresh
   - `POST /auth/logout` - revoke current session (requires auth)
   - `GET /auth/me` - return current user with roles (requires auth)
4. Implement `AuthService` with:
   - `validateUser(email, password): Promise<User | null>`
   - `login(user): Promise<LoginResponse>`
   - `refresh(refreshToken): Promise<RefreshTokenResponse>`
   - `logout(sessionId): Promise<void>`
   - `getProfile(userId): Promise<User>`
5. Add Swagger decorators matching openapi.yaml schemas

**Files**: `backend/src/auth/dto/`, `backend/src/auth/controllers/auth.controller.ts`, `backend/src/auth/services/auth.service.ts`

**Validation**:
- [ ] POST /auth/login returns 200 with accessToken, refreshToken, user
- [ ] Invalid credentials return 401 (same error for user not found/wrong password)
- [ ] POST /auth/refresh rotates refresh token (old revoked, new issued)
- [ ] POST /auth/logout revokes session, returns 204
- [ ] GET /auth/me returns user with roles array
- [ ] All endpoints documented in Swagger

---

### T008: Create RBAC middleware for 4 roles: customer, agent, supervisor, admin

**Purpose**: Role-based authorization per FR-002, FR-003.

**Steps**:
1. Create `src/common/decorators/roles.decorator.ts` - `@Roles(...roles)` decorator
2. Create `src/common/guards/roles.guard.ts` - RolesGuard:
   - Read required roles from reflector
   - Check user.roles includes at least one required role
   - Return 403 if insufficient permissions
3. Create `src/common/guards/jwt-auth.guard.ts` - extends AuthGuard('jwt')
4. Create `src/common/guards/jwt-refresh.guard.ts` - for refresh endpoint
5. Create `src/common/decorators/current-user.decorator.ts` - `@CurrentUser()`
5. Create `src/common/decorators/public.decorator.ts` - `@Public()` to skip auth
6. Register guards globally in AppModule or per-controller
7. Define role hierarchy constants: `ROLE_HIERARCHY = { admin: 4, supervisor: 3, agent: 2, customer: 1 }`

**Files**: `backend/src/common/guards/`, `backend/src/common/decorators/`

**Validation**:
- [ ] @Public() allows unauthenticated access
- [ ] @Roles('admin') blocks non-admins (403)
- [ ] @Roles('agent', 'supervisor', 'admin') allows any of those
- [ ] @CurrentUser() returns authenticated user
- [ ] Customer role cannot access agent endpoints

---

### T009: Implement session management with refresh token rotation/revocation

**Purpose**: Secure session handling per research.md risk register.

**Steps**:
1. Enhance `SessionService` from T006:
   - `rotateRefreshToken(oldTokenHash, newTokenHash): Promise<Session>` - atomic update
   - `isTokenRevoked(tokenHash): Promise<boolean>`
   - `getActiveSessions(userId): Promise<Session[]>`
2. Implement refresh token rotation in `AuthService.refresh()`:
   - Verify old refresh token
   - Generate new access + refresh tokens
   - Hash new refresh token
   - Atomically update session: revoke old, create new with new hash
   - Return new tokens
3. Add `revoked_at` timestamp on logout/rotation
4. Add cron job (via BullMQ later) to clean expired sessions daily
5. Add device tracking: user_agent, ip_address in Session

**Files**: `backend/src/auth/services/session.service.ts`, `backend/src/auth/services/auth.service.ts`

**Validation**:
- [ ] Refresh rotates token (old invalid, new valid)
- [ ] Concurrent refresh attempts handled (only one succeeds)
- [ ] Logout revokes session immediately
- [ ] Revoked token cannot be used again
- [ ] Multiple devices = multiple sessions

---

### T010: Add rate limiting on auth endpoints (5 attempts/15min per IP) [P]

**Purpose**: Prevent brute force per research.md risk register.

**Steps**:
1. Use @nestjs/throttler (already in WP01)
2. Create `src/auth/guards/auth-throttle.guard.ts` extending ThrottlerGuard:
   - Custom key generator: IP + endpoint
   - Limit: 5 requests per 15 minutes (900 seconds)
3. Apply `@UseGuards(AuthThrottleGuard)` to login, refresh endpoints
3. Configure ThrottlerModule in AppModule with custom storage (Redis for distributed)
4. Return 429 with `Retry-After` header on limit exceeded

**Files**: `backend/src/auth/guards/auth-throttle.guard.ts`

**Validation**:
- [ ] 6th login attempt within 15 min returns 429
- [ ] Retry-After header present
- [ ] Limit resets after 15 minutes
- [ ] Different IPs have independent limits

---

### T011: Write unit/integration tests for auth flow [P]

**Purpose**: Verify auth correctness per NFR-010.

**Steps**:
1. Unit tests for TokenService, PasswordService, SessionService
2. Integration tests for AuthController:
   - Login with valid credentials
   - Login with invalid credentials (401)
   - Refresh token rotation
   - Refresh with revoked token (401)
   - Logout revokes session
   - Me endpoint with valid token
   - Me endpoint without token (401)
   - Role guard enforcement
3. Use Testcontainers for PostgreSQL in integration tests
4. Mock Redis for throttler tests
5. Aim for >80% coverage on auth module

**Files**: `backend/src/auth/**/*.spec.ts`, `backend/test/auth/`

**Validation**:
- [ ] All unit tests pass
- [ ] All integration tests pass
- [ ] Coverage >80% for auth module
- [ ] Tests run in CI pipeline

---

## Branch Strategy

- **Planning Branch**: `epic1`
- **Work Package Branch**: `feature/wp02-auth-foundation` (created from `epic1`)
- **Final Merge Target**: `epic1`
- **Execution Worktree**: Allocated by `finalize-tasks` from `lanes.json`
- **Implementation Command**: `spec-kitty agent action implement WP02 --agent claude`

---

## Test Strategy

- Unit tests for all services (token, password, session)
- Integration tests for all endpoints using Supertest + Testcontainers
- Test JWT token generation/validation edge cases
- Test refresh token rotation concurrency
- Test rate limiting enforcement

---

## Definition of Done

- [ ] All 6 subtasks complete with validation passing
- [ ] Login/refresh/logout/me endpoints work per OpenAPI
- [ ] RBAC guards enforce 4 roles correctly
- [ ] Refresh token rotation implemented and tested
- [ ] Rate limiting active on auth endpoints
- [ ] Unit + integration tests pass (>80% coverage)
- [ ] Swagger documentation matches contracts/openapi.yaml
- [ ] No security vulnerabilities (Argon2id, short access tokens, rotation)

---

## Risks

- Token refresh race conditions - use database transactions for rotation
- Refresh token storage - hash with Argon2id, never log plain tokens
- Session management across SSE connections - sessions valid for SSE auth
- Rate limiting in distributed setup - use Redis-backed throttler storage

---

## Reviewer Guidance

- Verify Argon2id parameters meet OWASP recommendations
- Check JWT claims: sub, email, roles, iat, exp, jti
- Confirm refresh token rotation invalidates old token atomically
- Validate rate limiting uses IP + endpoint key, not just IP
- Ensure Swagger matches contracts/openapi.yaml exactly