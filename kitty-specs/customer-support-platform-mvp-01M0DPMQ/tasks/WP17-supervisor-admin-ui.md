---
work_package_id: WP17
title: Supervisor & Admin UI
dependencies:
- WP07
- WP09
- WP10
- WP14
requirement_refs:
- FR-064
- FR-081
- FR-090
- FR-096
tracker_refs: []
planning_base_branch: epic1
merge_target_branch: epic1
branch_strategy: Planning artifacts for this mission were generated on epic1. During /spec-kitty.implement this WP may branch from a dependency-specific base, but completed changes must merge back into epic1 unless the human explicitly redirects the landing branch.
subtasks:
- T064
- T065
- T066
agent: claude
history:
- timestamp: '2026-08-25T18:45:19Z'
  action: created
  actor: spec-kitty-tasks
agent_profile: implementer-ivan
authoritative_surface: frontend/src/pages/
create_intent:
- frontend/src/pages/SupervisorDashboard.tsx
- frontend/src/pages/AdminSettings.tsx
- frontend/src/pages/AdminStatuses.tsx
- frontend/src/pages/AdminPriorities.tsx
- frontend/src/pages/AdminCategories.tsx
- frontend/src/pages/AdminQueues.tsx
- frontend/src/pages/AdminSlaRules.tsx
- frontend/src/pages/AdminMacros.tsx
- frontend/src/pages/AdminAuditLog.tsx
execution_mode: code_change
model: claude-sonnet-4-6
owned_files:
- frontend/src/pages/SupervisorDashboard.tsx
- frontend/src/pages/AdminSettings.tsx
- frontend/src/pages/AdminStatuses.tsx
- frontend/src/pages/AdminPriorities.tsx
- frontend/src/pages/AdminCategories.tsx
- frontend/src/pages/AdminQueues.tsx
- frontend/src/pages/AdminSlaRules.tsx
- frontend/src/pages/AdminMacros.tsx
- frontend/src/pages/AdminAuditLog.tsx
role: implementer
tags: []
---

## Do This First: Load Agent Profile

```bash
/opencode/skill ad-hoc-profile-load --name implementer-ivan
```

---

## Objective

Build supervisor dashboards (workload, queue backlog, aging, SLA risk) and admin configuration UI (statuses, priorities, categories, queues, SLA rules, macros, audit log).

---

## Context

- **Dependencies**: WP07 (search/queues), WP09 (dashboards/reports), WP10 (admin config), WP14 (frontend setup)
- **Reference Documents**: spec.md (FR-064, FR-081, FR-090, FR-096), contracts/openapi.yaml (Reports, Admin, Audit tags)

---

## Detailed Guidance per Subtask

### T064: Build supervisor dashboard: workload, queue backlog, aging, SLA risk

**Purpose**: Supervisor visibility per FR-081, FR-064.

**Steps**:
1. Create `src/pages/SupervisorDashboard.tsx`:
   - Top metrics cards: total volume, open cases, overdue, unassigned
   - Workload by agent: table (agent, assigned, open, avg resolution)
   - Queue backlog: per queue (count, avg age hours, oldest case)
   - Aging buckets: horizontal bar chart or table (0-2h, 2-8h, 8-24h, 1-3d, 3-7d, 7d+)
   - SLA risk: cases approaching/breaching targets
   - Real-time updates via SSE
   - Drill-down: click metric to filtered case list
2. Create `src/services/reports.api.ts` - typed client for dashboard endpoints

**Files**: `frontend/src/pages/SupervisorDashboard.tsx`, `frontend/src/services/reports.api.ts`, `frontend/src/components/SupervisorCharts.tsx`

**Validation**:
- [ ] All 5 dashboard sections render
- [ ] Data loads from API
- [ ] Real-time updates work
- [ ] Drill-down to case list works
- [ ] Supervisor/admin only access

---

### T065: Create admin configuration UI: statuses, priorities, categories, queues [P]

**Purpose**: Admin config per FR-090.

**Steps**:
1. Create `src/pages/AdminSettings.tsx` - tabbed interface:
   - Statuses tab: list, create, edit, reorder, deactivate (not system)
   - Priorities tab: list, create, edit, reorder
   - Categories tab: hierarchical tree, create, edit, reorder, deactivate
   - Queues tab: list, create, edit, deactivate (not catch-all), SLA rule link
2. Forms with validation matching backend DTOs
3. Confirmation dialogs for deactivation (checks active cases)
4. Real-time: changes reflect immediately

**Files**: `frontend/src/pages/AdminSettings.tsx`, `frontend/src/pages/AdminStatuses.tsx`, `frontend/src/pages/AdminPriorities.tsx`, `frontend/src/pages/AdminCategories.tsx`, `frontend/src/pages/AdminQueues.tsx`, `frontend/src/services/admin.api.ts`

**Validation**:
- [ ] All 4 config types manageable
- [ ] System statuses protected
- [ ] Catch-all queue protected
- [ ] Deactivation blocked with explanation
- [ ] Hierarchical categories work

---

### T066: Add SLA rule builder, macro manager, audit log viewer [P]

**Purpose**: Advanced admin per FR-092, FR-051, FR-096.

**Steps**:
1. SLA Rule Builder (`AdminSlaRules.tsx`):
   - Form: name, description, category (optional), priority (optional), queue (optional)
   - Response/resolution targets (business minutes)
   - Business hours selector
   - Specificity explanation
2. Macro Manager (`AdminMacros.tsx`):
   - List with shortcut keys
   - Create/edit: name, template (textarea with variable hints), field updates, shortcut key
   - Preview expansion
3. Audit Log Viewer (`AdminAuditLog.tsx`):
   - Paginated table with filters (event type, actor, entity, date range)
   - Expand row for before/after JSON
   - Export to CSV

**Files**: `frontend/src/pages/AdminSlaRules.tsx`, `frontend/src/pages/AdminMacros.tsx`, `frontend/src/pages/AdminAuditLog.tsx`, `frontend/src/services/admin.api.ts`

**Validation**:
- [ ] SLA rule builder with specificity guidance
- [ ] Macro manager with preview
- [ ] Audit log with filters, pagination, before/after
- [ ] Admin only access

---

## Branch Strategy

- **Planning Branch**: `epic1`
- **Work Package Branch**: `feature/wp17-supervisor-admin-ui` (from `epic1`)
- **Final Merge Target**: `epic1`
- **Implementation Command**: `spec-kitty agent action implement WP17 --agent claude`

---

## Test Strategy

- Component tests for dashboard widgets
- Component tests for admin forms
- Test deactivation validation
- Test audit log pagination/filters

---

## Definition of Done

- [ ] Supervisor dashboard with all 5 sections
- [ ] Admin config UI for 4 basic types
- [ ] SLA rule builder
- [ ] Macro manager
- [ ] Audit log viewer
- [ ] Authorization enforced
- [ ] Tests pass

---

## Risks

- Complex dashboard data aggregation
- Config validation UX
- Audit log pagination/filtering
- Permission-aware UI

---

## Reviewer Guidance

- Verify supervisor sees all agents/queues
- Check admin deactivation shows blocking cases
- Confirm audit log shows before/after
- Test SLA rule specificity UI