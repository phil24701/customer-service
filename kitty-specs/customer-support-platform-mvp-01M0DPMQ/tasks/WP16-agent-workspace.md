---
work_package_id: WP16
title: Agent Workspace
dependencies:
- WP04
- WP05
- WP06
- WP14
requirement_refs:
- FR-030
- FR-032
- FR-039
- FR-040
- FR-051
- FR-052
- FR-053
tracker_refs: []
planning_base_branch: epic1
merge_target_branch: epic1
branch_strategy: feature/wp16-agent-workspace
subtasks:
- T062
- T063
agent: claude
history:
- timestamp: '2026-08-25T18:45:19Z'
  action: created
  actor: spec-kitty-tasks
agent_profile: implementer-ivan
authoritative_surface: frontend/src/pages/Agent
create_intent: []
execution_mode: code_change
model: claude-sonnet-4-6
owned_files:
- frontend/src/pages/AgentDashboard.tsx
- frontend/src/pages/AgentCaseList.tsx
- frontend/src/pages/AgentCaseDetail.tsx
- frontend/src/components/CaseEditor.tsx
- frontend/src/hooks/useCaseDraft.ts
- frontend/src/services/macros.api.ts
role: implementer
tags: []
---

## ⚡ Do This First: Load Agent Profile

```bash
/opencode/skill ad-hoc-profile-load --name implementer-ivan
```

---

## Objective

Build agent-facing workspace: queue views, case editor with collision warning, macro picker, keyboard shortcuts, single-action reply+pending.

---

## Context

- **Dependencies**: WP04 (cases), WP05 (communications, macros, collision), WP06 (agent ops), WP14 (frontend setup)
- **Reference Documents**: spec.md (FR-030, FR-032, FR-039, FR-040, FR-051, FR-052, FR-053), contracts/openapi.yaml (Cases, Communications, Macros tags)

---

## Detailed Guidance per Subtask

### T062: Build agent workspace: queue views, case editor, collision warning UI

**Purpose**: Agent case management per FR-030, FR-039.

**Steps**:
1. Create `src/pages/AgentDashboard.tsx`:
   - Queue tabs: each queue agent has access to
   - Case list per queue: columns (number, subject, customer, priority, status, assignee, age, SLA)
   - Sorting, pagination
   - Real-time: case count badges update via SSE
2. Create `src/pages/AgentCaseDetail.tsx`:
   - Case header with editable fields (status, priority, category, queue, assignee)
   - Communications thread: all (internal + customer)
   - Internal note toggle
   - Collision warning banner: "User X is drafting a reply" with lock countdown
   - Draft lock: acquire on focus editor, release on submit/blur/timeout
3. Create `src/components/CaseEditor.tsx`:
   - Rich text editor (simple textarea + toolbar)
   - Internal note checkbox
   - Macro picker dropdown
   - Attachment upload
   - Submit actions: "Send Reply", "Send & Set Pending Customer"
4. Create `src/hooks/useCaseDraft.ts`:
   - `acquireLock(caseId, type)` → POST /cases/:id/collision
   - `releaseLock(caseId)` → DELETE /cases/:id/collision
   - Auto-release on unmount, beforeunload (navigator.sendBeacon)
   - SSE listener for collision.warning/collision.released

**Files**: `frontend/src/pages/AgentDashboard.tsx`, `frontend/src/pages/AgentCaseDetail.tsx`, `frontend/src/components/CaseEditor.tsx`, `frontend/src/hooks/useCaseDraft.ts`, `frontend/src/services/collision.api.ts`

**Validation**:
- [ ] Queue views with case lists
- [ ] Case editor with all fields
- [ ] Collision warning appears when other user drafting
- [ ] Lock acquired on editor focus
- [ ] Lock released on submit/navigate/timeout
- [ ] Real-time queue updates

---

### T063: Implement macro picker, keyboard shortcuts, single-action reply+pending [P]

**Purpose**: Agent efficiency per FR-051, FR-052, FR-053, FR-040.

**Steps**:
1. Macro picker in CaseEditor:
   - Fetch macros: GET /macros (filtered by role)
   - Dropdown with name, description, shortcut key
   - On select: expand template, apply field updates
2. Keyboard shortcuts:
   - Global listener (Cmd/Ctrl + key)
   - Configurable per macro (shortcut_key)
   - Actions: submit reply, toggle reply/note, apply macro
   - Respect input focus (don't trigger in textareas)
3. Single-action reply + pending:
   - Button: "Send & Set Pending Customer"
   - Calls POST /cases/:id/respond (WP06/T029)
   - Creates communication + transitions status

**Files**: `frontend/src/components/MacroPicker.tsx`, `frontend/src/hooks/useKeyboardShortcuts.ts`, `frontend/src/components/CaseEditor.tsx` (integration)

**Validation**:
- [ ] Macro picker shows available macros
- [ ] Macro applies template + field updates
- [ ] Keyboard shortcuts work
- [ ] Shortcuts don't trigger in inputs
- [ ] Single-action reply+pending works

---

## Branch Strategy

- **Planning Branch**: `epic1`
- **Work Package Branch**: `feature/wp16-agent-workspace` (from `epic1`)
- **Final Merge Target**: `epic1`
- **Implementation Command**: `spec-kitty agent action implement WP16 --agent claude`

---

## Test Strategy

- Component tests for CaseEditor, MacroPicker
- Test collision warning UI
- Test keyboard shortcuts
- Integration test: agent workflow

---

## Definition of Done

- [ ] Queue views with real-time updates
- [ ] Case editor with collision warning
- [ ] Macro picker with field updates
- [ ] Keyboard shortcuts
- [ ] Single-action reply+pending
- [ ] Tests pass

---

## Risks

- Real-time collision UI race conditions
- Macro picker UX
- Keyboard shortcut conflicts
- Optimistic locking UI

---

## Reviewer Guidance

- Verify collision warning shows drafting user + countdown
- Check macro field updates validated
- Confirm shortcuts don't interfere with editing
- Test single-action reply+pending transition