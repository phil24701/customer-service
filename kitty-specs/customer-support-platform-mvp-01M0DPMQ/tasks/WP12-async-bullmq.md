---
work_package_id: WP12
title: Async Processing & Background Jobs (BullMQ)
dependencies:
- WP01
requirement_refs:
- C-007
- FR-073
- FR-100
tracker_refs: []
planning_base_branch: epic1
merge_target_branch: epic1
branch_strategy: Planning artifacts for this mission were generated on epic1. During /spec-kitty.implement this WP may branch from a dependency-specific base, but completed changes must merge back into epic1 unless the human explicitly redirects the landing branch.
subtasks:
- T051
- T052
- T053
agent: claude
history:
- timestamp: '2026-08-25T18:45:19Z'
  action: created
  actor: spec-kitty-tasks
agent_profile: implementer-ivan
authoritative_surface: backend/src/jobs/
create_intent:
- backend/src/config/queue.config.ts
- backend/src/services/queue.service.ts
execution_mode: code_change
model: claude-sonnet-4-6
owned_files:
- backend/src/jobs/**
- backend/src/config/queue.config.ts
- backend/src/services/queue.service.ts
role: implementer
tags: []
---

## ⚡ Do This First: Load Agent Profile

```bash
/opencode/skill ad-hoc-profile-load --name implementer-ivan
```

---

## Objective

Set up BullMQ (Redis-based) queues for all deferred work: SLA monitoring, email delivery, report generation, audit writes with retries, exponential backoff, dead letter queues, and priority queues.

---

## Context

- **Dependencies**: WP01 (Redis, BullMQ installed)
- **Reference Documents**: spec.md (C-007, FR-073, FR-100), plan.md (IC-13), research.md (Bull/BullMQ for Async Processing), DM-01M0DWBJ (Bull/BullMQ)

---

## Detailed Guidance per Subtask

### T051: Set up BullMQ queues: sla-monitor, email-delivery, audit-write, reports

**Purpose**: Queue infrastructure per DM-01M0DWBJ.

**Steps**:
1. Create `src/config/queue.config.ts`:
   - Redis connection config (host, port, password, db)
   - Queue definitions with default job options
2. Create `src/services/queue.service.ts`:
   - `getQueue(name): Queue` - singleton per queue
   - `addJob(queueName, jobName, data, opts?): Promise<Job>`
   - Queue names: `sla-monitor`, `email-delivery`, `email-dead-letter`, `audit-write`, `report-generation`, `case-indexing`
3. Create queue modules: `src/jobs/queues/` with each queue's processor registration
4. Register in AppModule

**Files**: `backend/src/config/queue.config.ts`, `backend/src/services/queue.service.ts`, `backend/src/jobs/queues/`

**Validation**:
- [ ] All queues created and connected to Redis
- [ ] Queue service provides typed access
- [ ] Jobs can be added to any queue

---

### T052: Configure job retries, exponential backoff, dead letter queues [P]

**Purpose**: Reliability per research.md.

**Steps**:
1. Default job options in queue config:
   - `attempts: 3`
   - `backoff: { type: 'exponential', delay: 1000 }` (1s, 2s, 4s)
   - `removeOnComplete: 100`
   - `removeOnFail: 50`
2. Dead letter queue pattern:
   - On job failure after max attempts: move to `{queueName}-dlq`
   - DLQ processor: log, alert admin, allow manual retry
3. Create `src/jobs/processors/base.processor.ts` with common error handling
4. Each queue's processor extends base

**Files**: `backend/src/config/queue.config.ts`, `backend/src/jobs/processors/base.processor.ts`, `backend/src/jobs/queues/*-dlq.processor.ts`

**Validation**:
- [ ] Jobs retry with exponential backoff
- [ ] Failed jobs go to DLQ after max attempts
- [ ] DLQ accessible for admin review
- [ ] Manual retry from DLQ works

---

### T053: Add priority queues for SLA breach alerts (high) vs reports (low) [P]

**Purpose**: Priority processing per research.md.

**Steps**:
1. BullMQ priority: lower number = higher priority
2. Queue priorities:
   - `sla-monitor`: priority 1 (highest)
   - `email-delivery`: priority 5
   - `email-dead-letter`: priority 3
   - `audit-write`: priority 10
   - `report-generation`: priority 20 (lowest)
3. Job options: `priority: number`
4. Worker concurrency: higher for priority queues

**Files**: `backend/src/config/queue.config.ts`, `backend/src/jobs/queues/`

**Validation**:
- [ ] SLA monitor jobs process before reports
- [ ] Priority respected under load
- [ ] Worker concurrency configured per queue

---

## Branch Strategy

- **Planning Branch**: `epic1`
- **Work Package Branch**: `feature/wp12-async-bullmq` (from `epic1`)
- **Final Merge Target**: `epic1`
- **Implementation Command**: `spec-kitty agent action implement WP12 --agent claude`

---

## Test Strategy

- Unit tests for queue service
- Integration tests for job processing
- Test retry/backoff behavior
- Test DLQ routing
- Test priority ordering under load

---

## Definition of Done

- [ ] All 6 queues configured
- [ ] Retries with exponential backoff
- [ ] Dead letter queues for all
- [ ] Priority levels working
- [ ] Tests pass
- [ ] Metrics exposed to Prometheus

---

## Risks

- Job idempotency - design for re-execution
- Redis connection resilience - retry connection
- Queue backlog monitoring - metrics
- Priority inversion - avoid
- Dead letter handling - don't lose jobs

---

## Reviewer Guidance

- Verify all queues have DLQ
- Check priority levels match requirements
- Confirm job options (attempts, backoff, removeOnComplete/Fail)
- Test graceful shutdown (wait for active jobs)