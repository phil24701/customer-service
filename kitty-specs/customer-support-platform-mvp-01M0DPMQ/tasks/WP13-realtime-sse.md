---
work_package_id: WP13
title: Real-Time Event Distribution (SSE)
dependencies:
- WP01
- WP05
- WP06
requirement_refs:
- FR-110
- FR-111
- NFR-011
tracker_refs: []
planning_base_branch: epic1
merge_target_branch: epic1
branch_strategy: feature/wp13-realtime-sse
subtasks:
- T054
- T055
- T056
agent: claude
history:
- timestamp: '2026-08-25T18:45:19Z'
  action: created
  actor: spec-kitty-tasks
agent_profile: implementer-ivan
authoritative_surface: backend/src/realtime/
create_intent: []
execution_mode: code_change
model: claude-sonnet-4-6
owned_files:
- backend/src/realtime/
- backend/src/sse/
role: implementer
tags: []
---

## ⚡ Do This First: Load Agent Profile

```bash
/opencode/skill ad-hoc-profile-load --name implementer-ivan
```

---

## Objective

Implement Server-Sent Events for server→client push with graceful degradation to optimistic locking, connection management with reconnection, and real-time events for case/queue updates and collision warnings.

---

## Context

- **Dependencies**: WP01 (observability), WP05 (collision), WP06 (case updates)
- **Reference Documents**: spec.md (FR-110, FR-111, NFR-011), plan.md (IC-11), contracts/openapi.yaml (Real-time tag), research.md (Real-Time Collision Prevention, SSE Event Format), DM-01M0DXQS (Hybrid SSE+REST)

---

## Detailed Guidance per Subtask

### T054: Implement graceful SSE degradation → optimistic locking fallback

**Purpose**: Graceful degradation per NFR-011, research.md.

**Steps**:
1. Create `src/realtime/services/realtime.service.ts`:
   - `emit(event: SSEEvent): void` - broadcast to connected clients
   - `getConnectionCount(): number`
2. Fallback mechanism:
   - Client tracks SSE connection state
   - On disconnect: switch to polling (every 30s) for case/queue updates
   - Optimistic locking: include `version` field on Case, increment on update
   - On mutation: if version mismatch → 409 Conflict, client refetches
3. Feature flag: `REALTIME_ENABLED` (default true)

**Files**: `backend/src/realtime/services/realtime.service.ts`, `backend/src/realtime/realtime.module.ts`

**Validation**:
- [ ] SSE emits events to connected clients
- [ ] Client detects disconnect
- [ ] Polling fallback activates
- [ ] Optimistic locking works on mutation conflict

---

### T055: Build SSE connection management with reconnection logic [P]

**Purpose**: Connection lifecycle per research.md.

**Steps**:
1. Create `src/sse/controllers/sse.controller.ts`:
   - `GET /events` - SSE endpoint (requires auth)
   - Headers: `Content-Type: text/event-stream`, `Cache-Control: no-cache`, `Connection: keep-alive`
   - Heartbeat: send `: heartbeat` every 30s
   - Query params: `channels=case,queue,collision,notification,sla`
2. Connection management:
   - Track connections per user (Map<userId, Set<EventSource>>)
   - Cleanup on disconnect
   - Max connections per user: 5
   - Reconnection: client uses EventSource (auto-reconnects)
   - Last-Event-ID for missed events (optional)

**Files**: `backend/src/sse/controllers/sse.controller.ts`, `backend/src/realtime/services/connection-manager.ts`

**Validation**:
- [ ] SSE endpoint streams events
- [ ] Heartbeat keeps connection alive
- [ ] Channels filter events
- [ ] Auto-reconnect works
- [ ] Max connections enforced

---

### T056: Create real-time events: case.updated, queue.changed, collision.warning [P]

**Purpose**: Event emission per FR-110, FR-111, contracts/openapi.yaml SSEEvent.

**Steps**:
1. Define event types in `src/realtime/events.ts`:
   - `case.created`, `case.updated`, `case.deleted`, `case.status_changed`, `case.assigned`
   - `queue.changed` (case count changes)
   - `collision.warning`, `collision.released`
   - `notification.new`
   - `sla.breach_warning`, `sla.breach`
2. Emit from relevant services:
   - CaseService: on create/update/delete/status/assign
   - QueueService: on case enter/leave queue
   - CollisionService: on acquire/release
   - SlaMonitorJob: on breach
3. Event format per contracts/openapi.yaml SSEEvent:
   - event, data, timestamp, correlationId

**Files**: `backend/src/realtime/events.ts`, integration in CaseService, QueueService, CollisionService, SlaMonitorJob

**Validation**:
- [ ] All event types emitted correctly
- [ ] Event format matches OpenAPI
- [ ] Channels work (client subscribes to subset)
- [ ] Correlation ID for tracing
- [ ] <100ms p95 delivery (perf goal)

---

## Branch Strategy

- **Planning Branch**: `epic1`
- **Work Package Branch**: `feature/wp13-realtime-sse` (from `epic1`)
- **Final Merge Target**: `epic1`
- **Implementation Command**: `spec-kitty agent action implement WP13 --agent claude`

---

## Test Strategy

- Unit tests for event emission
- Integration test for SSE endpoint
- Test reconnection and missed events
- Test fallback to polling
- Load test: 10k concurrent connections

---

## Definition of Done

- [ ] SSE endpoint with auth, channels, heartbeat
- [ ] Connection management with reconnection
- [ ] All event types emitted
- [ ] Graceful degradation to polling + optimistic locking
- [ ] Tests pass
- [ ] Swagger matches contracts

---

## Risks

- Connection management at scale - use Redis pub/sub for multi-instance
- Event ordering guarantees - include sequence numbers
- Reconnection logic - EventSource handles, but test
- Fallback to polling - ensure no duplicate updates
- Memory leaks - cleanup on disconnect

---

## Reviewer Guidance

- Verify EventSource auto-reconnect works
- Check optimistic locking version field on Case
- Confirm all FR-110/FR-111 events emitted
- Test multi-instance with Redis pub/sub