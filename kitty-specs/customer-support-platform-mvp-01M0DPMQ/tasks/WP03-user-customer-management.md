---
work_package_id: WP03
title: User & Customer Management
dependencies:
- WP02
requirement_refs:
- FR-004
- FR-005
- FR-006
- FR-010
- FR-011
- FR-012
- FR-013
- FR-014
tracker_refs: []
planning_base_branch: epic1
merge_target_branch: epic1
branch_strategy: feature/wp03-user-customer-management
subtasks:
- T012
- T013
- T014
- T015
- T016
agent: claude
history:
- timestamp: '2026-08-25T18:45:19Z'
  action: created
  actor: spec-kitty-tasks
agent_profile: implementer-ivan
authoritative_surface: backend/src/users/
create_intent: []
execution_mode: code_change
model: claude-sonnet-4-6
owned_files:
- backend/src/users/
- backend/src/customers/
- backend/src/organizations/
- backend/prisma/schema.prisma
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

Implement user administration (admin-only CRUD), customer management with search and case history, organization support, and seed realistic demo data for portfolio demonstration.

---

## Context

- **Mission**: customer-support-platform-mvp-01M0DPMQ
- **Phase**: Epic 1 completion
- **Dependencies**: WP02 (auth, RBAC working)
- **Reference Documents**: spec.md (FR-004, FR-005, FR-006, FR-010 through FR-014), plan.md (IC-02), data-model.md (User, UserRole, Customer, Organization), contracts/openapi.yaml (Users, Customers, Organizations tags), research.md (PII handling, search performance)

---

## Detailed Guidance per Subtask

### T012: Create User, UserRole, Session, Tenant Prisma models and migrations

**Purpose**: Database models for user management (extends WP01/T002).

**Steps**:
1. Verify Prisma schema from WP01 includes all User, UserRole, Session, Tenant fields per data-model.md
2. Create migration for any missing fields: `npx prisma migrate dev --name user-management`
3. Add PrismaClient extensions for soft delete (is_active filtering)
4. Create `src/users/repositories/user.repository.ts` with:
   - `findByEmail(tenantId, email): Promise<User | null>`
   - `findById(id): Promise<User | null>`
   - `create(data): Promise<User>`
   - `update(id, data): Promise<User>`
   - `softDelete(id): Promise<void>`
   - `findMany(filters, pagination): Promise<[User[], number]>`
5. Create similar repositories for UserRole, Session

**Files**: `backend/prisma/schema.prisma`, `backend/prisma/migrations/`, `backend/src/users/repositories/`

**Validation**:
- [ ] Migration applies cleanly
- [ ] Repositories handle soft delete (is_active = false)
- [ ] Unique constraints on email per tenant
- [ ] Session refresh_token_hash unique index works

---

### T013: Build user management CRUD endpoints (admin only)

**Purpose**: Admin user management per FR-004, FR-005, FR-006.

**Steps**:
1. Create `src/users/dto/create-user.dto.ts`, `update-user.dto.ts` with Zod validation
2. Create `src/users/controllers/users.controller.ts`:
   - `GET /users` - paginated list with filters (role, isActive, search) - @Roles('admin')
   - `POST /users` - create user with roles - @Roles('admin')
   - `GET /users/:id` - get user by ID - @Roles('admin')
   - `PATCH /users/:id` - update user (name, roles, isActive) - @Roles('admin')
   - `DELETE /users/:id` - deactivate user (soft delete) - @Roles('admin')
3. Create `src/users/services/users.service.ts` with business logic:
   - Password hashing on create
   - Role assignment validation (admin can assign any role)
   - Audit logging on all changes (FR-006)
   - Prevent self-deactivation
4. Add Swagger documentation matching openapi.yaml Users tag

**Files**: `backend/src/users/dto/`, `backend/src/users/controllers/`, `backend/src/users/services/`, `backend/src/users/users.module.ts`

**Validation**:
- [ ] Admin can list users with pagination/filters
- [ ] Admin can create user with roles
- [ ] Admin can update user roles and status
- [ ] Admin cannot deactivate self
- [ ] Deactivation is soft delete (is_active = false)
- [ ] Audit logs created for all changes
- [ ] Non-admin gets 403

---

### T014: Create Customer, Organization models and customer search/list endpoints [P]

**Purpose**: Customer management per FR-010, FR-011, FR-012, FR-014.

**Steps**:
1. Verify Prisma schema includes Customer, Organization per data-model.md
2. Create `src/customers/repositories/customer.repository.ts`:
   - `search(tenantId, query: { name?, email?, organizationId?, externalId? }): Promise<Customer[]>`
   - `findById(id): Promise<Customer | null>`
   - `findWithCaseHistory(customerId): Promise<Customer & { cases: Case[] }>`
   - `create(data): Promise<Customer>`
   - `update(id, data): Promise<Customer>`
3. Create `src/organizations/repositories/organization.repository.ts`
4. Create `src/customers/controllers/customers.controller.ts`:
   - `GET /customers` - search with pagination - @Roles('agent', 'supervisor', 'admin')
   - `POST /customers` - create customer - @Roles('agent', 'supervisor', 'admin')
   - `GET /customers/:id` - customer with case history - @Roles('agent', 'supervisor', 'admin')
   - `PATCH /customers/:id` - update customer - @Roles('agent', 'supervisor', 'admin')
5. Create `src/organizations/controllers/organizations.controller.ts`:
   - `GET /organizations`, `POST /organizations`, `GET /organizations/:id`, `PATCH /organizations/:id` - @Roles('admin')

**Files**: `backend/src/customers/`, `backend/src/organizations/`

**Validation**:
- [ ] Customer search by name, email, organization, external_id works
- [ ] Customer profile shows case history
- [ ] Organization CRUD for admins
- [ ] Pagination and sorting work
- [ ] Agent/supervisor can access, customer role cannot

---

### T015: Implement customer profile with case history view

**Purpose**: Customer self-service portal support per FR-011, FR-013.

**Steps**:
1. Create `GET /customers/me` endpoint for authenticated customers:
   - Returns customer's own profile
   - Includes their cases with status, subject, created_at
2. Create `GET /customers/me/cases` - paginated case list for customer
3. Add authorization: customer can only access own data (FR-013)
4. Use existing case filtering but scoped to customer_id

**Files**: `backend/src/customers/controllers/customers.controller.ts`, `backend/src/customers/services/customers.service.ts`

**Validation**:
- [ ] Customer sees only own profile
- [ ] Customer sees only own cases
- [ ] Case list shows status, subject, dates
- [ ] Pagination works

---

### T016: Seed demo data: tenant, admin, agents, supervisor, customers, org [P]

**Purpose**: Realistic demo data per NFR-008.

**Steps**:
1. Create `prisma/seed.ts` with comprehensive seed:
   - 1 Tenant: "Demo Corp" (slug: demo)
   - 1 Admin: admin@demo.com / password123
   - 2 Agents: agent1@demo.com, agent2@demo.com
   - 1 Supervisor: supervisor@demo.com
   - 3 Customer users: customer1@demo.com, etc.
   - 1 Organization: "Acme Inc" (domain: acme.com)
   - 3 External customers (no user account)
   - 6 System Statuses (new, open, pending_customer, pending_internal, resolved, closed)
   - 4 Priorities (low, medium, high, critical)
   - 5 Categories (Technical, Billing, Account, General, Feature Request)
   - 4 Queues (General, Technical, Billing, Catch-All)
   - 3 SLA Rules (Critical: 15min/4hr, High: 1hr/8hr, Default: 4hr/24hr)
   - 1 Business Hours (Mon-Fri 9-5 UTC)
   - 5 Macros (common responses with shortcuts)
   - 20 Cases across statuses/priorities with communications
2. Run `npx prisma db seed` to populate
3. Verify all seed data accessible via API

**Files**: `backend/prisma/seed.ts`

**Validation**:
- [ ] `npx prisma db seed` completes without errors
- [ ] All demo users can login
- [ ] Cases exist in all statuses
- [ ] SLA rules configured
- [ ] Macros have shortcut keys
- [ ] Catch-all queue exists and is immutable

---

## Branch Strategy

- **Planning Branch**: `epic1`
- **Work Package Branch**: `feature/wp03-user-customer-management` (from `epic1`)
- **Final Merge Target**: `epic1`
- **Execution Worktree**: Allocated by `finalize-tasks`
- **Implementation Command**: `spec-kitty agent action implement WP03 --agent claude`

---

## Test Strategy

- Unit tests for repositories
- Integration tests for all endpoints
- Test authorization boundaries (customer vs agent vs admin)
- Test search performance with indexes
- Seed data verification tests

---

## Definition of Done

- [ ] All 5 subtasks complete
- [ ] User CRUD (admin only) works with audit logging
- [ ] Customer search/list/create/update works for agents+
- [ ] Customer self-service shows own cases only
- [ ] Organization CRUD for admins
- [ ] Demo seed data populates all required entities
- [ ] Tests pass (>80% coverage)
- [ ] Swagger matches contracts/openapi.yaml

---

## Risks

- PII handling in customer data - ensure logs don't expose emails/names
- Search performance at scale - verify indexes used
- Organization relationship integrity - foreign key constraints
- Self-deactivation prevention - admin cannot lock themselves out

---

## Reviewer Guidance

- Verify admin endpoints require admin role (403 for others)
- Check customer data isolation (FR-013)
- Confirm audit logs on all admin changes (FR-006)
- Validate seed data matches NFR-008 requirements exactly
- Ensure Prisma soft delete pattern consistent across models