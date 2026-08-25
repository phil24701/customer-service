---
work_package_id: WP04
title: Case Lifecycle Engine (Core)
dependencies:
- WP03
requirement_refs:
- FR-020
- FR-021
- FR-022
- FR-023
- FR-024
- FR-025
- FR-026
- FR-034
- FR-035
- FR-036
- FR-070
- FR-071
- FR-072
- FR-100
- FR-101
- FR-102
- FR-103
tracker_refs: []
planning_base_branch: epic1
merge_target_branch: epic1
branch_strategy: feature/wp04-case-lifecycle
subtasks:
- T017
- T018
- T019
- T020
- T021
- T022
agent: claude
history:
- timestamp: '2026-08-25T18:45:19Z'
  action: created
  actor: spec-kitty-tasks
agent_profile: implementer-ivan
authoritative_surface: backend/src/cases/
create_intent: []
execution_mode: code_change
model: claude-sonnet-4-6
owned_files:
- backend/src/cases/
- backend/src/routing/
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

Implement the core case lifecycle: state machine with 6 statuses, routing engine with catch-all fallback, SLA target calculation, case CRUD, and comprehensive audit/activity logging.

---

## Context

- **Mission**: customer-support-platform-mvp-01M0DPMQ
- **Phase**: Core platform (post-Epic 1)
- **Dependencies**: WP03 (users, customers, config seeded)
- **Reference Documents**: spec.md (FR-020 through FR-041, FR-070 through FR-075, FR-100 through FR-103), plan.md (IC-03, IC-07, IC-10), data-model.md (Case, Status, Priority, Category, Queue, SLARule, BusinessHours, CaseActivity, AuditLog), contracts/openapi.yaml (Cases, SLA tags), research.md (Case State Machine Invariants, SLA Pause/Resume Edge Cases, Configuration Referential Integrity)

---

## Detailed Guidance per Subtask

### T017: Create Case, Status, Priority, Category, Queue, SLARule, BusinessHours models

**Purpose**: Database models for case lifecycle (extends WP01/T002).

**Steps**:
1. Verify Prisma schema from WP01 includes all Case, Status, Priority, Category, Queue, SLARule, BusinessHours fields per data-model.md
2. Key Case fields: case_number (human-readable CSP-YYYY-NNNNNN), sla_paused_at, sla_total_paused_duration, email_message_id, email_thread_id
3. Status: pauses_sla boolean (only pending_customer=true), requires_root_cause (only resolved=true), is_system
4. Queue: is_catch_all (immutable), auto_assign
5. Create migration: `npx prisma migrate dev --name case-lifecycle`
6. Create repositories for each entity with tenant-scoped queries

**Files**: `backend/prisma/schema.prisma`, `backend/prisma/migrations/`, `backend/src/cases/repositories/`, `backend/src/routing/repositories/`

**Validation**:
- [ ] Migration applies cleanly
- [ ] Case number generation works (unique per tenant)
- [ ] Status pauses_sla only true for pending_customer
- [ ] Catch-all queue has is_catch_all=true

---

### T018: Implement case state machine with transition guards (6 statuses)

**Purpose**: Enforce valid status transitions per spec.

**Steps**:
1. Create `src/cases/services/case-state-machine.ts`:
   ```typescript
   const VALID_TRANSITIONS: Record<string, string[]> = {
     'new': ['open'],
     'open': ['pending_customer', 'pending_internal', 'resolved'],
     'pending_customer': ['open', 'resolved'],
     'pending_internal': ['open', 'resolved'],
     'resolved': ['closed'],
     'closed': []
   }
   ```
2. Implement `canTransition(from, to): boolean`
3. Implement `transition(case, toStatus, actor, context): Promise<Case>`:
   - Validate transition allowed
   - Check guards: to RESOLVED requires root_cause, category, final_priority
   - Check guards: to PENDING_INTERNAL requires internal_owner_id OR internal_department
   - Update status_id, updated_at
   - Handle SLA pause/resume (pending_customer pauses, open resumes)
   - Create AuditLog entry (FR-102)
   - Create CaseActivity entry (FR-035)
   - Emit SSE event (case.status_changed)
4. Create `src/cases/guards/transition.guard.ts` for controller validation

**Files**: `backend/src/cases/services/case-state-machine.ts`, `backend/src/cases/guards/transition.guard.ts`

**Validation**:
- [ ] Invalid transitions rejected (e.g., new → closed)
- [ ] RESOLVED requires root_cause, category, final_priority
- [ ] PENDING_INTERNAL requires internal owner/dept
- [ ] SLA pauses in pending_customer, resumes in open
- [ ] Audit log created on every status change

---

### T019: Build case routing engine with catch-all queue fallback & admin alert

**Purpose**: Automatic routing per FR-023.

**Steps**:
1. Create `src/routing/services/routing-engine.ts`:
   - `routeCase(input: { categoryId?, priorityId?, customerId? }): Promise<CaseRoutingResult>`
   - Evaluation order: category+priority+queue rules → category+priority → category → priority → queue → default
   - On failure or deprecated config: return catch-all queue ID + warning
   - Generate admin alert (AuditLog event_type: 'routing.failed')
2. Create `src/routing/services/catch-all.service.ts`:
   - Ensure exactly one catch-all queue per tenant (seed creates it)
   - Immutable: cannot deactivate/delete (DB constraint + app check)
3. Integrate into case creation flow

**Files**: `backend/src/routing/services/routing-engine.ts`, `backend/src/routing/services/catch-all.service.ts`

**Validation**:
- [ ] Routing follows specificity hierarchy
- [ ] Failure routes to catch-all queue
- [ ] Admin alert generated on routing failure
- [ ] Catch-all queue cannot be deactivated/deleted

---

### T020: Create case CRUD endpoints with validation per OpenAPI

**Purpose**: REST API for case management per contracts/openapi.yaml Cases tag.

**Steps**:
1. Create DTOs: `CreateCaseDto`, `UpdateCaseDto`, `CaseBulkActionDto` matching openapi.yaml
2. Create `src/cases/controllers/cases.controller.ts`:
   - `POST /cases` - create case (customer or agent) - routing engine called
   - `GET /cases` - list with filters (status, priority, category, queue, agent, date, search)
   - `GET /cases/:id` - case details with communications, SLA targets
   - `PATCH /cases/:id` - update case (status, priority, category, queue, assignment)
   - `POST /cases/bulk` - bulk actions (WP06/T032)
3. Create `src/cases/services/cases.service.ts` with business logic
4. Validation: subject 1-500, description 1-10000, customer must exist/active
5. Defaults: status=new, priority=first active, queue=routing result

**Files**: `backend/src/cases/dto/`, `backend/src/cases/controllers/`, `backend/src/cases/services/`, `backend/src/cases/cases.module.ts`

**Validation**:
- [ ] Case creation generates human-readable number (CSP-2026-000123)
- [ ] Routing engine called on creation
- [ ] Filters work: statusIds, priorityIds, categoryIds, queueIds, agentIds, customerId, dateFrom, dateTo, slaBreachRisk, overdue, unassigned, search
- [ ] Pagination and sorting work
- [ ] Customer role can only create/view own cases

---

### T021: Implement SLA target calculation on case creation (business hours) [P]

**Purpose**: Calculate response/resolution targets per FR-070, FR-071.

**Steps**:
1. Create `src/sla/services/sla-calculator.ts`:
   - `calculateTargets(case, slaRule): SLATargets`
   - Use BusinessHours for business minutes only
   - Exclude holidays
   - Account for existing paused duration
   - Return responseTargetAt, resolutionTargetAt, isPaused
2. On case creation: find matching SLARule (most specific), calculate targets, save to case
3. On status change to pending_customer: pause timer (save paused_at)
4. On status change from pending_customer to open: resume (add to total_paused_duration)
5. Store targets on case: sla_target_response_at, sla_target_resolution_at, sla_paused_at, sla_total_paused_duration

**Files**: `backend/src/sla/services/sla-calculator.ts`, `backend/src/cases/services/cases.service.ts` (integration)

**Validation**:
- [ ] Targets calculated on creation using matching SLA rule
- [ ] Business hours respected (Mon-Fri 9-5 UTC default)
- [ ] Holidays excluded
- [ ] Pause/resume correctly updates targets
- [ ] Targets stored on case for querying

---

### T022: Add case activity log (denormalized) and full audit trail [P]

**Purpose**: Audit requirements per FR-100, FR-101, FR-102, FR-103.

**Steps**:
1. Create `src/audit/services/audit.service.ts`:
   - `log(event: AuditEventInput): Promise<AuditLog>`
   - Event types per data-model.md (case.*, communication.*, user.*, config.*, etc.)
   - Include before/after JSONB for changes
   - Actor info: id, role, IP, user-agent
2. Create `src/cases/services/case-activity.service.ts`:
   - `log(caseId, actorId, type, summary, metadata): Promise<CaseActivity>`
   - Types: status_change, assignment, comment, priority_change, category_change, escalation, etc.
   - Denormalized for performance (FR-035)
3. Integrate into all case mutations (create, update, status change, assignment, communication)
4. AuditLog: immutable (no update/delete), indexed for query performance
5. Admin endpoint: `GET /audit` with filters (WP10)

**Files**: `backend/src/audit/services/audit.service.ts`, `backend/src/cases/services/case-activity.service.ts`

**Validation**:
- [ ] Every assignment creates audit log (FR-102)
- [ ] Every status change creates audit log (FR-102)
- [ ] Audit records include event_type, actor, timestamp, before/after
- [ ] CaseActivity denormalized for fast case timeline
- [ ] Normal users cannot alter/delete audit history (FR-103)

---

## Branch Strategy

- **Planning Branch**: `epic1`
- **Work Package Branch**: `feature/wp04-case-lifecycle` (from `epic1`)
- **Final Merge Target**: `epic1`
- **Execution Worktree**: Allocated by `finalize-tasks`
- **Implementation Command**: `spec-kitty agent action implement WP04 --agent claude`

---

## Test Strategy

- Unit tests for state machine transitions (all valid/invalid combos)
- Unit tests for routing engine (specificity, fallback, catch-all)
- Unit tests for SLA calculator (business hours, holidays, pause/resume)
- Integration tests for case CRUD endpoints
- Test audit logging on all mutations
- Test case number generation uniqueness

---

## Definition of Done

- [ ] All 6 subtasks complete
- [ ] Case state machine enforces all 6 statuses with correct guards
- [ ] Routing engine with catch-all fallback and admin alerts
- [ ] Case CRUD endpoints match OpenAPI contracts
- [ ] SLA targets calculated on creation with business hours
- [ ] SLA pause/resume works on pending_customer ↔ open transitions
- [ ] Full audit trail + denormalized activity log
- [ ] Tests pass (>80% coverage)
- [ ] Swagger matches contracts/openapi.yaml

---

## Risks

- State transition bugs - exhaustive test matrix needed
- SLA timer accuracy during pause/resume - persist paused_at, total_paused_duration
- Routing rule evaluation failures - must not silently lose cases (catch-all + alert)
- Email parsing edge cases - handled in WP11
- Configuration change impact on in-flight SLA timers - don't retroactively alter

---

## Reviewer Guidance

- Verify every transition in state matrix tested
- Check catch-all queue immutability (DB constraint + app check)
- Validate SLA calculator against BusinessHours + holidays
- Confirm audit logs immutable (no UPDATE/DELETE in API)
- Ensure case_number format: CSP-YYYY-NNNNNN (sequential per tenant)