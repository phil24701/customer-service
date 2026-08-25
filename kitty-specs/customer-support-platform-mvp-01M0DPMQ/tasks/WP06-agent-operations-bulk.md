---
work_package_id: WP06
title: Agent Case Operations & Bulk Actions
dependencies:
- WP04
- WP05
requirement_refs:
- FR-030
- FR-031
- FR-032
- FR-033
- FR-034
- FR-035
- FR-036
- FR-037
- FR-038
- FR-040
- FR-041
- FR-065
- FR-066
tracker_refs: []
planning_base_branch: epic1
merge_target_branch: epic1
branch_strategy: Planning artifacts for this mission were generated on epic1. During /spec-kitty.implement this WP may branch from a dependency-specific base, but completed changes must merge back into epic1 unless the human explicitly redirects the landing branch.
subtasks:
- T028
- T029
- T030
- T031
- T032
agent: claude
history:
- timestamp: '2026-08-25T18:45:19Z'
  action: created
  actor: spec-kitty-tasks
agent_profile: implementer-ivan
authoritative_surface: backend/src/cases/services/case-operations.service.ts
create_intent:
- backend/src/cases/services/case-operations.service.ts
- backend/src/cases/controllers/cases.controller.ts
execution_mode: code_change
model: claude-sonnet-4-6
owned_files:
- backend/src/cases/services/case-operations.service.ts
- backend/src/cases/controllers/cases.controller.ts
- backend/src/escalation/**
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

Implement agent-facing case operations: assignment/reassignment, status transitions with auto pending_customer→open, pending_internal requiring owner/department, escalation as distinct event, and bulk actions (≤100 cases) with same auth rules.

---

## Context

- **Mission**: customer-support-platform-mvp-01M0DPMQ
- **Dependencies**: WP04 (case lifecycle), WP05 (communications, collision)
- **Reference Documents**: spec.md (FR-030 through FR-041, FR-065, FR-066), plan.md (IC-04, IC-06), data-model.md (Case fields: assigned_agent_id, internal_owner_id, internal_department), contracts/openapi.yaml (Cases bulk, status transitions), research.md (Real-Time Collision Prevention, SLA Pause/Resume Edge Cases)

---

## Detailed Guidance per Subtask

### T028: Create case assignment/reassignment with supervisor override

**Purpose**: Case assignment per FR-031, FR-037.

**Steps**:
1. Create `src/cases/services/case-operations.service.ts`:
   - `assignCase(caseId, agentId, actorId, actorRole): Promise<Case>`
   - `reassignCase(caseId, newAgentId, actorId, actorRole): Promise<Case>`
   - Normal users: can only assign to agents in same queue(s)
   - Supervisors (@Roles('supervisor', 'admin')): can override, assign to any agent
   - Update assigned_agent_id, create AuditLog (case.assigned), CaseActivity
   - Release any collision lock for this case
   - Emit SSE case.updated
2. Add validation: agent must be active, have access to case's queue

**Files**: `backend/src/cases/services/case-operations.service.ts`, `backend/src/cases/controllers/cases.controller.ts`

**Validation**:
- [ ] Agent can assign to queue members
- [ ] Supervisor can assign to any agent
- [ ] Audit log created on assignment
- [ ] Collision lock released on reassignment
- [ ] SSE event emitted

---

### T029: Build status transitions with auto Pending Customer → Open on inbound

**Purpose**: Status transitions per FR-032, FR-034, FR-040.

**Steps**:
1. Extend `CaseStateMachine` (WP04/T018) with agent operations:
   - `sendResponseAndSetPendingCustomer(caseId, communicationDto, agentId): Promise<Case>`
     - Creates customer-visible communication
     - Transitions case to pending_customer (single action FR-040)
     - Pauses SLA timer
   - Inbound customer communication handling (called from WP11 email or WP05 communications):
     - If case.status == pending_customer → transition to open
     - Resumes SLA timer
2. Controller endpoints:
   - `POST /cases/:id/respond` - agent sends reply + sets pending_customer
   - `PATCH /cases/:id/status` - general status change (validated by state machine)

**Files**: `backend/src/cases/services/case-state-machine.ts`, `backend/src/cases/services/case-operations.service.ts`, `backend/src/cases/controllers/cases.controller.ts`

**Validation**:
- [ ] Agent can send response + set pending_customer in one call
- [ ] Inbound customer communication auto-opens pending_customer cases
- [ ] SLA pauses on pending_customer, resumes on open
- [ ] Transition to resolved requires root_cause, category, final_priority

---

### T030: Implement Pending Internal requiring internal owner/department

**Purpose**: Internal handoff per FR-041.

**Steps**:
1. Add to `CaseStateMachine`:
   - `escalateToInternal(caseId, internalOwnerId?, internalDepartment, agentId): Promise<Case>`
   - Requires internalOwnerId XOR internalDepartment (not both, not neither)
   - Sets internal_owner_id OR internal_department
   - Status → pending_internal
   - SLA continues running (only pending_customer pauses)
   - Create AuditLog (case.escalated), CaseActivity
2. Controller: `POST /cases/:id/escalate` with body { internalOwnerId?, internalDepartment }
3. Resolution from pending_internal: same guards as open→resolved (needs root_cause, category, final_priority)

**Files**: `backend/src/cases/services/case-state-machine.ts`, `backend/src/cases/services/case-operations.service.ts`

**Validation**:
- [ ] Escalation requires owner XOR department
- [ ] Status becomes pending_internal
- [ ] SLA continues running (not paused)
- [ ] Audit log with escalation event
- [ ] Can resolve from pending_internal with required fields

---

### T031: Add escalation as distinct operational event with audit [P]

**Purpose**: Escalation tracking per FR-038, FR-075.

**Steps**:
1. Escalation is distinct from reassignment - it's a handoff to another department
2. Create `src/escalation/services/escalation.service.ts`:
   - `recordEscalation(caseId, fromAgentId, toDepartment, reason): Promise<EscalationRecord>`
   - Separate from case status change - can escalate while staying in open
   - But typically: open → pending_internal = escalation
3. AuditLog event_type: 'case.escalated' with metadata { fromAgent, toDepartment, reason }
4. Supervisor dashboard (WP09) shows escalation metrics
5. CaseActivity: activity_type='escalation', summary='Escalated to {department}'

**Files**: `backend/src/escalation/services/escalation.service.ts`, `backend/src/escalation/escalation.module.ts`

**Validation**:
- [ ] Escalation recorded as distinct event
- [ ] Audit log captures from/to/reason
- [ ] Case activity shows escalation
- [ ] Supervisor can view escalation history

---

### T032: Create bulk actions: assign, status, priority, category, queue (≤100 cases)

**Purpose**: Bulk operations per FR-065, FR-066.

**Steps**:
1. Create `src/cases/services/bulk-operations.service.ts`:
   - `bulkAction(caseIds, action, params, actorId, actorRole): Promise<BulkResult>`
   - Actions: assign, status_change, priority_change, category_change, queue_change
   - Max 100 cases per request
   - Process sequentially (not parallel) to maintain audit trail order
   - For each case: validate auth, validate transition, execute, log audit
   - Return { successCount, failedCount, errors[] }
2. Controller: `POST /cases/bulk` (already in WP04/T020)
3. Auth: same rules as individual actions (FR-066)
   - Bulk assign: supervisor can override, agent limited to queue
   - Bulk status: validate each transition
   - Bulk priority/category/queue: validate each

**Files**: `backend/src/cases/services/bulk-operations.service.ts`, `backend/src/cases/controllers/cases.controller.ts`

**Validation**:
- [ ] Bulk assign respects auth rules per case
- [ ] Bulk status validates each transition
- [ ] Max 100 cases enforced
- [ ] Partial success reported with errors
- [ ] Audit log for each successful case
- [ ] Same business rules as individual actions

---

## Branch Strategy

- **Planning Branch**: `epic1`
- **Work Package Branch**: `feature/wp06-agent-ops-bulk` (from `epic1`)
- **Final Merge Target**: `epic1`
- **Execution Worktree**: Allocated by `finalize-tasks`
- **Implementation Command**: `spec-kitty agent action implement WP06 --agent claude`

---

## Test Strategy

- Unit tests for each operation type
- Integration tests for bulk endpoint
- Test supervisor override vs agent restrictions
- Test partial failure handling
- Test audit logging for bulk operations
- Test collision lock release on bulk reassignment

---

## Definition of Done

- [ ] All 5 subtasks complete
- [ ] Assignment/reassignment with supervisor override
- [ ] Status transitions with auto pending_customer→open
- [ ] Pending internal requires owner/department
- [ ] Escalation as distinct audited event
- [ ] Bulk actions (≤100) with per-case auth validation
- [ ] Tests pass (>80% coverage)
- [ ] Swagger matches contracts/openapi.yaml

---

## Risks

- Real-time collision prevention race conditions on bulk reassignment
- Bulk action authorization consistency - each case validated individually
- Optimistic locking conflicts - use database transactions per case
- Escalation audit trail completeness - separate from status change

---

## Reviewer Guidance

- Verify bulk actions enforce same auth as individual (FR-066)
- Check supervisor override works for assignment
- Validate pending_internal requires owner XOR department
- Confirm escalation recorded separately from status change
- Ensure bulk operations atomic per case (not all-or-nothing)