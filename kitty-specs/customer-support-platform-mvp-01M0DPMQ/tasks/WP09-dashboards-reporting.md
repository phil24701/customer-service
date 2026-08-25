---
work_package_id: WP09
title: Dashboards & Reporting
dependencies:
- WP04
- WP07
- WP08
requirement_refs:
- FR-080
- FR-081
- FR-082
- FR-083
- FR-084
- FR-085
tracker_refs: []
planning_base_branch: epic1
merge_target_branch: epic1
branch_strategy: feature/wp09-dashboards-reporting
subtasks:
- T038
- T039
- T040
- T041
agent: claude
history:
- timestamp: '2026-08-25T18:45:19Z'
  action: created
  actor: spec-kitty-tasks
agent_profile: implementer-ivan
authoritative_surface: backend/src/reports/
create_intent: []
execution_mode: code_change
model: claude-sonnet-4-6
owned_files:
- backend/src/reports/
- backend/src/dashboard/
role: implementer
tags: []
---

## ⚡ Do This First: Load Agent Profile

```bash
/opencode/skill ad-hoc-profile-load --name implementer-ivan
```

---

## Objective

Implement agent/supervisor dashboards with volume reporting by status/priority/category/time, and metrics (avg response/resolution, SLA compliance) with drill-down.

---

## Context

- **Dependencies**: WP04 (Case data), WP07 (Search/filtering), WP08 (SLA targets)
- **Reference Documents**: spec.md (FR-080 through FR-085), plan.md (IC-08), contracts/openapi.yaml (Reports tag), research.md (Query performance on large datasets)

---

## Detailed Guidance per Subtask

### T038: Build agent dashboard: assigned/open/attention cases, queue counts

**Purpose**: Agent dashboard per FR-080.

**Steps**:
1. Create `src/dashboard/services/agent-dashboard.service.ts`:
   - `getDashboard(agentId): Promise<AgentDashboard>`
   - assignedCases: count where assigned_agent_id = agentId AND status NOT closed
   - openCases: count where assigned_agent_id = agentId AND status = open
   - needsAttention: overdue + SLA risk + pending_customer (awaiting customer)
   - myQueueCounts: for each queue agent has access to, count cases
2. Controller: `GET /dashboard/agent`
3. Authorization: @Roles('agent', 'supervisor', 'admin'), scoped to agent

**Files**: `backend/src/dashboard/services/agent-dashboard.service.ts`, `backend/src/dashboard/controllers/`

**Validation**:
- [ ] Counts accurate for assigned agent
- [ ] Needs attention includes overdue, SLA risk, pending_customer
- [ ] Queue counts per accessible queue
- [ ] Real-time updates via SSE (WP13)

---

### T039: Build supervisor dashboard: volume, workload, backlog, aging, overdue [P]

**Purpose**: Supervisor dashboard per FR-081.

**Steps**:
1. Create `src/dashboard/services/supervisor-dashboard.service.ts`:
   - `getDashboard(): Promise<SupervisorDashboard>`
   - totalVolume: cases created in period (default 30 days)
   - workloadByAgent: per agent (assigned, open counts)
   - queueBacklog: per queue (count, avgAgeHours)
   - aging: buckets (0-2h, 2-8h, 8-24h, 1-3d, 3-7d, 7d+)
   - overdueCases: count where sla_target_resolution_at < now()
2. Controller: `GET /dashboard/supervisor`
3. Authorization: @Roles('supervisor', 'admin')

**Files**: `backend/src/dashboard/services/supervisor-dashboard.service.ts`, `backend/src/dashboard/controllers/`

**Validation**:
- [ ] Volume by time period
- [ ] Workload per agent
- [ ] Queue backlog with avg age
- [ ] Aging buckets correct
- [ ] Overdue count accurate

---

### T040: Implement volume reporting by status/priority/category/time period [P]

**Purpose**: Volume reports per FR-082, FR-084.

**Steps**:
1. Create `src/reports/services/volume-report.service.ts`:
   - `getVolumeReport(filters): Promise<VolumeReport>`
   - Group by: status, priority, category, time period (day/week/month)
   - Filters: dateFrom, dateTo, queueIds, agentIds
   - Drill-down: each group links to filtered case list
2. Controller: `GET /reports/volume`
3. Authorization: @Roles('supervisor', 'admin'), scoped

**Files**: `backend/src/reports/services/volume-report.service.ts`, `backend/src/reports/controllers/`

**Validation**:
- [ ] Grouping by all 4 dimensions works
- [ ] Time periods: day, week, month
- [ ] Drill-down to case list works
- [ ] Authorization scoping

---

### T041: Add metrics: avg response/resolution, SLA compliance, drill-down to cases [P]

**Purpose**: Metrics per FR-083, FR-084.

**Steps**:
1. Create `src/reports/services/metrics-report.service.ts`:
   - `getMetricsReport(filters): Promise<MetricsReport>`
   - avgResponseTimeMinutes: cases with first response / response time
   - avgResolutionTimeMinutes: resolved cases / resolution time
   - slaComplianceRate: resolved within target / total resolved
   - Business hours only
2. Controller: `GET /reports/metrics`
3. Drill-down: each metric links to filtered case list

**Files**: `backend/src/reports/services/metrics-report.service.ts`, `backend/src/reports/controllers/`

**Validation**:
- [ ] Avg response time (first agent reply)
- [ ] Avg resolution time (created → resolved)
- [ ] SLA compliance rate
- [ ] Business hours only
- [ ] Drill-down works

---

## Branch Strategy

- **Planning Branch**: `epic1`
- **Work Package Branch**: `feature/wp09-dashboards-reporting` (from `epic1`)
- **Final Merge Target**: `epic1`
- **Implementation Command**: `spec-kitty agent action implement WP09 --agent claude`

---

## Test Strategy

- Unit tests for aggregation queries
- Integration tests for dashboard endpoints
- Test authorization scoping
- Performance test with large datasets

---

## Definition of Done

- [ ] Agent dashboard with all metrics
- [ ] Supervisor dashboard with all metrics
- [ ] Volume reports by 4 dimensions
- [ ] Metrics: response, resolution, SLA compliance
- [ ] Drill-down to case lists
- [ ] Authorization scoping
- [ ] Tests pass
- [ ] Swagger matches contracts

---

## Risks

- Query performance on large datasets - use materialized views or caching
- Real-time dashboard updates - SSE integration
- Authorization scoping on reports
- Metric calculation accuracy

---

## Reviewer Guidance

- Verify business hours used for metrics
- Check drill-down preserves filters
- Confirm authorization scoping (agent sees own, supervisor sees all)
- Test with edge cases (no data, single case)