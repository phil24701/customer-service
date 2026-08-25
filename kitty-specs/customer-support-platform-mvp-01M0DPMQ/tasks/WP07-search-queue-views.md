---
work_package_id: WP07
title: Search, Filtering & Queue Views
dependencies:
- WP04
requirement_refs:
- FR-060
- FR-061
- FR-062
- FR-063
- FR-064
tracker_refs: []
planning_base_branch: epic1
merge_target_branch: epic1
branch_strategy: Planning artifacts for this mission were generated on epic1. During /spec-kitty.implement this WP may branch from a dependency-specific base, but completed changes must merge back into epic1 unless the human explicitly redirects the landing branch.
subtasks:
- T033
- T034
- T035
agent: claude
history:
- timestamp: '2026-08-25T18:45:19Z'
  action: created
  actor: spec-kitty-tasks
agent_profile: implementer-ivan
authoritative_surface: backend/src/search/
create_intent: []
execution_mode: code_change
model: claude-sonnet-4-6
owned_files:
- backend/src/search/**
- backend/src/queues/**
role: implementer
tags: []
---

## ⚡ Do This First: Load Agent Profile

```bash
/opencode/skill ad-hoc-profile-load --name implementer-ivan
```

---

## Objective

Implement PostgreSQL full-text search, multi-dimensional filtering, queue views with pagination, and supervisor views (unassigned, overdue, SLA risk).

---

## Context

- **Dependencies**: WP04 (Case model with search_vector)
- **Reference Documents**: spec.md (FR-060 through FR-066), plan.md (IC-06), data-model.md (Case search_vector, indexes), contracts/openapi.yaml (Search, Queues tags), research.md (PostgreSQL Full-Text Search)

---

## Detailed Guidance per Subtask

### T033: Build search with PostgreSQL full-text (tsvector/GIN) and multi-filter

**Purpose**: Case search per FR-060, FR-061.

**Steps**:
1. Create `src/search/services/search.service.ts`:
   - `searchCases(tenantId, query: SearchQuery): Promise<PaginatedResult<Case>>`
   - Full-text: `to_tsquery('english', search)` against Case.search_vector
   - Ranking: `ts_rank_cd(search_vector, query)` with weights (subject=A, description=B, customer=C, case_number=A)
   - Filters: statusIds, priorityIds, categoryIds, queueIds, assignedAgentIds, customerId, dateFrom, dateTo, slaBreachRisk, overdue, unassigned
   - Combine: WHERE (fulltext OR filters) AND tenant_id
   - Pagination: offset/limit with total count
2. Trigger to update search_vector on case/communication insert/update:
   ```sql
   CREATE TRIGGER update_search_vector
   BEFORE INSERT OR UPDATE ON cases
   FOR EACH ROW EXECUTE FUNCTION tsvector_update_trigger();
   ```
3. Controller: `GET /search/cases` (or `GET /cases` with search param)

**Files**: `backend/src/search/services/search.service.ts`, `backend/src/search/controllers/`, `backend/prisma/migrations/` (trigger)

**Validation**:
- [ ] Full-text search returns ranked results
- [ ] Filters combine with AND logic
- [ ] Pagination works with total count
- [ ] Search includes subject, description, customer name, case number
- [ ] GIN index used (EXPLAIN ANALYZE)

---

### T034: Create queue views with case counts, sorting, pagination [P]

**Purpose**: Queue views per FR-062, FR-063.

**Steps**:
1. Create `src/queues/services/queue.service.ts`:
   - `getQueuesWithCounts(tenantId): Promise<QueueWithCount[]>`
   - `getQueueCases(queueId, filters, pagination): Promise<PaginatedResult<Case>>`
   - Counts: total, by status (new, open, pending_customer, etc.)
2. Controller: `GET /queues`, `GET /queues/:id/cases`
3. Sorting: created_at, priority, sla_target_resolution_at, updated_at
4. Real-time: SSE queue.changed when case enters/leaves queue

**Files**: `backend/src/queues/services/queue.service.ts`, `backend/src/queues/controllers/`

**Validation**:
- [ ] Queue list shows case counts per status
- [ ] Queue case view paginated and sortable
- [ ] Real-time updates via SSE
- [ ] Agent sees only accessible queues

---

### T035: Implement supervisor views: unassigned, overdue, SLA risk cases [P]

**Purpose**: Supervisor visibility per FR-064.

**Steps**:
1. Add to `QueueService` or new `SupervisorService`:
   - `getUnassignedCases(tenantId, pagination): Promise<PaginatedResult<Case>>` - status != closed, assigned_agent_id IS NULL
   - `getOverdueCases(tenantId, pagination): Promise<PaginatedResult<Case>>` - sla_target_resolution_at < now()
   - `getSlaRiskCases(tenantId, thresholdMinutes, pagination): Promise<PaginatedResult<Case>>` - target within threshold
2. Controller: `GET /supervisor/unassigned`, `GET /supervisor/overdue`, `GET /supervisor/sla-risk`
3. Authorization: @Roles('supervisor', 'admin')

**Files**: `backend/src/supervisor/services/supervisor.service.ts`, `backend/src/supervisor/controllers/`

**Validation**:
- [ ] Unassigned cases filter works
- [ ] Overdue uses sla_target_resolution_at
- [ ] SLA risk configurable threshold
- [ ] Supervisor/admin only access
- [ ] Pagination and sorting work

---

## Branch Strategy

- **Planning Branch**: `epic1`
- **Work Package Branch**: `feature/wp07-search-queues` (from `epic1`)
- **Final Merge Target**: `epic1`
- **Implementation Command**: `spec-kitty agent action implement WP07 --agent claude`

---

## Test Strategy

- Unit tests for search query building
- Integration tests for search + filters
- Test queue view counts accuracy
- Test supervisor view authorization
- Performance test with 10k+ cases

---

## Definition of Done

- [ ] Full-text search with ranking
- [ ] Multi-filter combination
- [ ] Queue views with counts
- [ ] Supervisor views: unassigned, overdue, SLA risk
- [ ] Tests pass
- [ ] Swagger matches contracts

---

## Risks

- PostgreSQL full-text search performance - verify GIN index usage
- Filter combination complexity - test edge cases
- Pagination consistency with real-time updates

---

## Reviewer Guidance

- Check search_vector trigger updates on communication insert
- Verify search weights (subject=A, description=B, customer=C, case_number=A)
- Confirm supervisor endpoints require supervisor/admin role