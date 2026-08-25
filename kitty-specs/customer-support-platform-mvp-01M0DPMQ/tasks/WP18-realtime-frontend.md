---
work_package_id: WP18
title: Real-Time Frontend Integration
dependencies:
- WP13
- WP15
- WP16
- WP17
requirement_refs:
- FR-110
- FR-111
- NFR-011
tracker_refs: []
planning_base_branch: epic1
merge_target_branch: epic1
branch_strategy: Planning artifacts for this mission were generated on epic1. During /spec-kitty.implement this WP may branch from a dependency-specific base, but completed changes must merge back into epic1 unless the human explicitly redirects the landing branch.
subtasks:
- T067
- T068
agent: claude
history:
- timestamp: '2026-08-25T18:45:19Z'
  action: created
  actor: spec-kitty-tasks
agent_profile: implementer-ivan
authoritative_surface: frontend/src/hooks/
create_intent:
- frontend/src/hooks/useRealtime.ts
- frontend/src/hooks/useCaseDraft.ts
- frontend/src/contexts/RealtimeContext.tsx
- frontend/src/services/sse.client.ts
execution_mode: code_change
model: claude-sonnet-4-6
owned_files:
- frontend/src/hooks/useRealtime.ts
- frontend/src/hooks/useCaseDraft.ts
- frontend/src/contexts/RealtimeContext.tsx
- frontend/src/services/sse.client.ts
role: implementer
tags: []
---

## Do This First: Load Agent Profile

```bash
/opencode/skill ad-hoc-profile-load --name implementer-ivan
```

---

## Objective

Connect all frontend mutation operations to SSE for live updates: case/queue changes reflect without refresh, collision warnings appear, graceful degradation works.

---

## Context

- **Dependencies**: WP13 (SSE backend), WP15 (customer portal), WP16 (agent workspace), WP17 (supervisor/admin)
- **Reference Documents**: spec.md (FR-110, FR-111, NFR-011), contracts/openapi.yaml (Real-time tag), research.md (SSE Event Format)

---

## Detailed Guidance per Subtask

### T067: Implement real-time hooks: useRealtime, useCaseDraft, EventSource wrapper

**Purpose**: Reusable real-time infrastructure.

**Steps**:
1. Create `frontend/src/services/sse.client.ts`:
   - `createEventSource(url, token): EventSource` - with auth header via query param
   - Auto-reconnect with exponential backoff
   - Handle `last-event-id` for missed events
   - Parse SSE format: event, data, timestamp, correlationId
2. Create `frontend/src/contexts/RealtimeContext.tsx`:
   - Single EventSource connection per session
   - Channel subscription management
   - Event distribution to subscribers
   - Connection state: connecting, connected, disconnected, reconnecting
3. Create `frontend/src/hooks/useRealtime.ts`:
   - `useRealtime(channel, eventType, handler)` - subscribe to events
   - `useRealtimeConnection()` - connection state
4. Create `frontend/src/hooks/useCaseDraft.ts` (enhance WP16):
   - Integrate with RealtimeContext for collision.warning/collision.released
   - Show warning banner when other user drafting

**Files**: `frontend/src/services/sse.client.ts`, `frontend/src/contexts/RealtimeContext.tsx`, `frontend/src/hooks/useRealtime.ts`, `frontend/src/hooks/useCaseDraft.ts`

**Validation**:
- [ ] Single SSE connection shared across app
- [ ] Channel subscription works
- [ ] Events dispatched to handlers
- [ ] Reconnection works
- [ ] Collision warnings via SSE

---

### T068: Add SSE integration to all mutation operations for live updates [P]

**Purpose**: Live updates per FR-110, FR-111.

**Steps**:
1. Case mutations (create, update, status change, assignment):
   - Invalidate React Query cache on case.updated
   - Or: optimistically update cache with SSE data
2. Queue views:
   - On queue.changed: refresh case counts
3. Dashboards:
   - On case/queue/sla events: refresh metrics
4. Communications:
   - On communication.created: append to thread
5. Collision:
   - On collision.warning: show banner
   - On collision.released: hide banner
6. Graceful degradation:
   - If SSE disconnected: show indicator, enable polling fallback
   - Polling: refetch queries every 30s

**Files**: Integration in `frontend/src/pages/`, `frontend/src/hooks/`, `frontend/src/services/`

**Validation**:
- [ ] Case changes reflect without refresh
- [ ] Queue counts update live
- [ ] Dashboard metrics update
- [ ] Communications appear in real-time
- [ ] Collision warnings work
- [ ] Polling fallback when SSE down

---

## Branch Strategy

- **Planning Branch**: `epic1`
- **Work Package Branch**: `feature/wp18-realtime-frontend` (from `epic1`)
- **Final Merge Target**: `epic1`
- **Implementation Command**: `spec-kitty agent action implement WP18 --agent claude`

---

## Test Strategy

- Test SSE connection lifecycle
- Test event handling for each type
- Test graceful degradation
- Test reconnection with missed events

---

## Definition of Done

- [ ] RealtimeContext with EventSource wrapper
- [ ] useRealtime hook for subscriptions
- [ ] All mutations trigger live updates
- [ ] Collision warnings via SSE
- [ ] Polling fallback works
- [ ] Tests pass

---

## Risks

- EventSource reconnection
- Memory leaks from subscriptions
- Event ordering
- UI update flicker

---

## Reviewer Guidance

- Verify single SSE connection shared
- Check EventSource auto-reconnect
- Confirm all FR-110/FR-111 events handled
- Test SSE down -> polling fallback