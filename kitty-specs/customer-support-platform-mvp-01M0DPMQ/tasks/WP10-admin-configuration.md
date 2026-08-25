---
work_package_id: WP10
title: Administration & Configuration
dependencies:
- WP04
requirement_refs:
- FR-090
- FR-091
- FR-092
- FR-093
- FR-094
- FR-095
- FR-096
tracker_refs: []
planning_base_branch: epic1
merge_target_branch: epic1
branch_strategy: Planning artifacts for this mission were generated on epic1. During /spec-kitty.implement this WP may branch from a dependency-specific base, but completed changes must merge back into epic1 unless the human explicitly redirects the landing branch.
subtasks:
- T042
- T043
- T044
- T045
- T046
agent: claude
history:
- timestamp: '2026-08-25T18:45:19Z'
  action: created
  actor: spec-kitty-tasks
agent_profile: implementer-ivan
authoritative_surface: backend/src/admin/
create_intent: []
execution_mode: code_change
model: claude-sonnet-4-6
owned_files:
- backend/src/admin/**
role: implementer
tags: []
---

## ⚡ Do This First: Load Agent Profile

```bash
/opencode/skill ad-hoc-profile-load --name implementer-ivan
```

---

## Objective

Implement full admin configuration: statuses, priorities, categories, queues, SLA rules, macros with referential integrity enforcement and immutable catch-all queue.

---

## Context

- **Dependencies**: WP04 (config models exist)
- **Reference Documents**: spec.md (FR-090 through FR-096), plan.md (IC-09), data-model.md (config entities), contracts/openapi.yaml (Admin tag), research.md (Configuration Referential Integrity)

---

## Detailed Guidance per Subtask

### T042: Create admin config endpoints: statuses, priorities, categories, queues

**Purpose**: Config management per FR-090.

**Steps**:
1. Create `src/admin/services/config.service.ts` with generic CRUD for each entity
2. Create controllers for each:
   - `StatusController`: CRUD + sort_order, is_system protection
   - `PriorityController`: CRUD + sort_order
   - `CategoryController`: CRUD + hierarchical (parent_id)
   - `QueueController`: CRUD + sla_rule_id, auto_assign
3. Validation: cannot delete system statuses (is_system=true)
4. Audit logging on all changes (FR-096)

**Files**: `backend/src/admin/services/config.service.ts`, `backend/src/admin/controllers/`, `backend/src/admin/admin.module.ts`

**Validation**:
- [ ] All 4 config types CRUD
- [ ] System statuses protected
- [ ] Hierarchical categories work
- [ ] Queue SLA rule linking
- [ ] Audit logs on all changes

---

### T043: Implement SLA rule management CRUD [P]

**Purpose**: SLA config per FR-092.

**Steps**:
1. Extend `SlaRuleService` (WP08/T036) with admin endpoints
2. Controller: `GET/POST/PATCH/DELETE /admin/sla-rules`
3. Validation: specificity matching, at least one dimension
4. Business hours linking

**Files**: `backend/src/admin/controllers/sla-rules.controller.ts`

**Validation**:
- [ ] SLA rules CRUD
- [ ] Specificity validation
- [ ] Business hours linking

---

### T044: Add macro management CRUD with shortcut keys [P]

**Purpose**: Macro config per FR-090, FR-053.

**Steps**:
1. Extend `MacroService` (WP05/T026) with admin endpoints
2. Controller: `GET/POST/PATCH/DELETE /admin/macros`
3. Shortcut key uniqueness per tenant
4. Variable documentation in description

**Files**: `backend/src/admin/controllers/macros.controller.ts`

**Validation**:
- [ ] Macros CRUD
- [ ] Shortcut keys unique
- [ ] Variables documented

---

### T045: Enforce referential integrity: prevent deletion of referenced config

**Purpose**: Config protection per FR-093, FR-094.

**Steps**:
1. Add to `ConfigService`:
   - `canDelete(entityType, entityId): Promise<boolean>`
   - Check: active cases referencing this config
   - For status/category/queue/priority: active = status NOT closed
2. On delete attempt: if referenced → 409 with details
3. Soft delete pattern: is_active = false instead of hard delete
4. Hard delete only if zero references

**Files**: `backend/src/admin/services/config.service.ts`

**Validation**:
- [ ] Referenced config cannot be deleted (409)
- [ ] Soft delete works
- [ ] Hard delete only when zero refs
- [ ] Error message shows what references exist

---

### T046: Make catch-all queue immutable (cannot deactivate/delete) [P]

**Purpose**: Catch-all protection per FR-095.

**Steps**:
1. Add DB constraint: `CHECK (NOT is_catch_all OR is_active)` - cannot deactivate
2. Add Prisma middleware: reject delete/update where is_catch_all=true
3. Admin controller: filter out catch-all from deletable list
4. Seed creates exactly one catch-all per tenant

**Files**: `backend/prisma/schema.prisma` (constraint), `backend/src/admin/services/queue.service.ts`

**Validation**:
- [ ] Catch-all queue cannot be deactivated
- [ ] Catch-all queue cannot be deleted
- [ ] Exactly one per tenant
- [ ] Admin UI hides delete/deactivate for catch-all

---

## Branch Strategy

- **Planning Branch**: `epic1`
- **Work Package Branch**: `feature/wp10-admin-config` (from `epic1`)
- **Final Merge Target**: `epic1`
- **Implementation Command**: `spec-kitty agent action implement WP10 --agent claude`

---

## Test Strategy

- Unit tests for referential integrity checks
- Integration tests for all config CRUD
- Test catch-all immutability
- Test soft delete vs hard delete

---

## Definition of Done

- [ ] All config types CRUD
- [ ] SLA rules, macros manageable
- [ ] Referential integrity enforced
- [ ] Catch-all queue immutable
- [ ] Audit logs on all changes
- [ ] Tests pass
- [ ] Swagger matches contracts

---

## Risks

- Referential integrity enforcement on config deletion
- Catch-all queue immutability - DB constraint + app check
- Permission matrix complexity
- Configuration change propagation

---

## Reviewer Guidance

- Verify 409 on delete of referenced config
- Check catch-all has DB constraint + app protection
- Confirm audit logs for all config changes
- Test hierarchical category CRUD