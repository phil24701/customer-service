---
work_package_id: WP08
title: SLA Management & Monitoring
dependencies:
- WP04
- WP12
requirement_refs:
- FR-070
- FR-071
- FR-072
- FR-073
- FR-074
- FR-075
tracker_refs: []
planning_base_branch: epic1
merge_target_branch: epic1
branch_strategy: feature/wp08-sla-management
subtasks:
- T036
- T037
agent: claude
history:
- timestamp: '2026-08-25T18:45:19Z'
  action: created
  actor: spec-kitty-tasks
agent_profile: implementer-ivan
authoritative_surface: backend/src/sla/
create_intent: []
execution_mode: code_change
model: claude-sonnet-4-6
owned_files:
- backend/src/sla/
- backend/src/jobs/sla-monitor.job.ts
- backend/prisma/schema.prisma
role: implementer
tags: []
---

## ⚡ Do This First: Load Agent Profile

```bash
/opencode/skill ad-hoc-profile-load --name implementer-ivan
```

---

## Objective

Implement configurable SLA rules by category/priority/queue with business hours, and BullMQ background job for breach detection and escalation events.

---

## Context

- **Dependencies**: WP04 (Case, SLA calculator), WP12 (BullMQ queues)
- **Reference Documents**: spec.md (FR-070 through FR-075), plan.md (IC-07, IC-13), data-model.md (SLARule, BusinessHours), contracts/openapi.yaml (SLA tag), research.md (SLA Pause/Resume Edge Cases, Bull/BullMQ for Async Processing)

---

## Detailed Guidance per Subtask

### T036: Add SLA configuration: rules by category/priority/queue with business hours

**Purpose**: SLA rule management per FR-070, FR-071.

**Steps**:
1. Create `src/sla/services/sla-rule.service.ts`:
   - CRUD for SLARule (admin only)
   - Matching logic: most specific wins (category+priority+queue > category+priority > category > priority > queue > default)
   - Validate: at least one of category/priority/queue specified
2. Create `src/sla/services/business-hours.service.ts`:
   - CRUD for BusinessHours (admin)
   - `isBusinessHour(date, businessHours): boolean`
   - `addBusinessMinutes(date, minutes, businessHours): Date`
   - Holidays excluded
3. Controller: `GET/POST/PATCH/DELETE /admin/sla-rules`, `/admin/business-hours`
4. Integration: Case creation uses SlaCalculator (WP04/T021) to find matching rule and calculate targets

**Files**: `backend/src/sla/services/`, `backend/src/sla/controllers/`, `backend/src/sla/sla.module.ts`

**Validation**:
- [ ] SLA rules CRUD for admins
- [ ] Matching specificity works correctly
- [ ] Business hours calculations correct (Mon-Fri 9-5 UTC default)
- [ ] Holidays excluded
- [ ] Targets calculated on case creation

---

### T037: Implement SLA monitor background job (BullMQ) for breach detection

**Purpose**: Breach detection per FR-073, FR-074, FR-075.

**Steps**:
1. Create `src/jobs/sla-monitor.job.ts`:
   - Queue: `sla-monitor` (high priority)
   - Schedule: every 5 minutes (cron: `*/5 * * * *`)
   - Job: find cases where sla_target_resolution_at < now() + threshold (e.g., 30 min)
   - For each case: create AuditLog (sla.breach_warning or sla.breach)
   - Create CaseActivity
   - Emit SSE event (sla.breach_warning, sla.breach)
2. Create `src/sla/services/sla-monitor.service.ts`:
   - `checkBreaches(): Promise<BreachResult[]>`
   - `checkApproaching(thresholdMinutes): Promise<Case[]>`
3. BullMQ job processor in `src/jobs/processors/sla-monitor.processor.ts`
4. Register queue in WP12 BullMQ setup

**Files**: `backend/src/jobs/sla-monitor.job.ts`, `backend/src/jobs/processors/sla-monitor.processor.ts`, `backend/src/sla/services/sla-monitor.service.ts`

**Validation**:
- [ ] Job runs every 5 minutes
- [ ] Detects approaching breaches (configurable threshold)
- [ ] Detects actual breaches
- [ ] Creates audit log events
- [ ] Emits SSE events
- [ ] High priority queue processing
- [ ] Idempotent (same breach not alerted repeatedly)

---

## Branch Strategy

- **Planning Branch**: `epic1`
- **Work Package Branch**: `feature/wp08-sla-management` (from `epic1`)
- **Final Merge Target**: `epic1`
- **Implementation Command**: `spec-kitty agent action implement WP08 --agent claude`

---

## Test Strategy

- Unit tests for SLA rule matching specificity
- Unit tests for business hours calculations (holidays, weekends)
- Integration test for monitor job
- Test breach detection accuracy

---

## Definition of Done

- [ ] SLA rules with specificity matching
- [ ] Business hours with holidays
- [ ] BullMQ monitor job detects approaching/actual breaches
- [ ] Audit logs + SSE events on breach
- [ ] Tests pass
- [ ] Swagger matches contracts

---

## Risks

- Timer accuracy across restarts - persist paused state
- Timezone handling - use UTC internally, business hours in tenant TZ
- Breach detection latency - job runs every 5 min
- Config change impact on in-flight cases - don't retroactively alter

---

## Reviewer Guidance

- Verify specificity matching: cat+pri+queue > cat+pri > cat > pri > queue > default
- Check business hours calculations with holidays
- Confirm monitor job idempotency
- Ensure high priority queue for SLA alerts