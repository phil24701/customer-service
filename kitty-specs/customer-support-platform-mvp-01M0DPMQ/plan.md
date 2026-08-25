# Implementation Plan: Customer Support Platform MVP

**Branch**: `epic1` | **Date**: 2026-08-19 | **Spec**: `kitty-specs/customer-support-platform-mvp-01M0DPMQ/spec.md`
**Input**: Feature specification from `/kitty-specs/customer-support-platform-mvp-01M0DPMQ/spec.md`

**Note**: This template is filled in by the `/spec-kitty.plan` command. See `src/doctrine/missions/software-dev/command-templates/plan.md` for the execution workflow.

The planner will not begin until all planning questions have been answered—capture those answers in this document before progressing to later phases.

## Summary

Build a production-style customer support case management platform with full case lifecycle, email ingestion, routing, SLA management, real-time collaboration, and role-based administration. Implementation phased starting with Epic 1 (Authentication & Access). Full MVP covers 17 validated items including authentication, case lifecycle, email-to-case, routing/queues, SLA, real-time updates, macros, bulk actions, audit logging, and admin configuration.

## Technical Context

<!--
  ACTION REQUIRED: Replace the content in this section with the technical details
  for the project. The structure here is presented in advisory capacity to guide
  the iteration process.

  If multiple developers/agents will work on this mission, add an "Implementation
  Concern Map" section below to decompose architectural intent into IC-## concerns
  before generating tasks.
-->

**Language/Version**: TypeScript 5.x (Node.js 20+ LTS)
**Primary Dependencies**: React 18+, NestJS (@nestjs/core, @nestjs/common, @nestjs/platform-express, @nestjs/jwt, @nestjs/passport), Prisma, Redis client (ioredis), BullMQ, Zod (validation)
**Storage**: PostgreSQL 15+ (primary), Redis 7+ (caching/transient state/sessions)
**Testing**: Unit (Vitest), Integration (Supertest + Testcontainers), E2E (Playwright)
**Target Platform**: Linux server (Docker containers), modern desktop browsers (Chrome, Firefox, Safari, Edge)
**Project Type**: Web application (frontend + backend)
**Performance Goals**: <200ms p95 API latency, <100ms p95 real-time event delivery, support 10k concurrent users
**Constraints**: Layered architecture (NFR-001), server-side validation/auth (NFR-002), referential integrity (NFR-003), externalized secrets (NFR-007), reproducible deploy (NFR-006)
**Scale/Scope**: ~50 screens, ~100 API endpoints, 17 functional areas, multi-role RBAC
**Async Mechanism**: Bull/BullMQ (Redis-based) — mature queue with retries, scheduling, priority, metrics, and horizontal scaling
**Real-Time Mechanism**: Hybrid — SSE for server→client push (case/queue updates, notifications) + REST for client→server mutations
**Search Implementation**: PostgreSQL full-text search (built-in, no extra infrastructure) — leverages existing PostgreSQL with tsvector/tsquery, GIN indexes
**Email Provider**: Postmark (free Developer plan for portfolio demo, isolated behind provider-independent boundary)
**CI/CD Platform**: GitHub Actions
**Observability Stack**: Pino + Prometheus + OpenTelemetry
**Attachment Storage**: S3-compatible (MinIO for local development/demo, configurable endpoint for production S3-compatible provider)
**Container Orchestration**: Both - Docker Compose for local development and Kubernetes manifests for live portfolio demonstration

## Charter Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

No charter file found at `.kittify/charter/charter.md`. Charter Check skipped — no governance constraints to validate against.

## Project Structure

### Documentation (this mission)

```
kitty-specs/[###-mission]/
├── plan.md              # This file (/spec-kitty.plan command output)
├── research.md          # Phase 0 output (/spec-kitty.plan command)
├── data-model.md        # Phase 1 output (/spec-kitty.plan command)
├── quickstart.md        # Phase 1 output (/spec-kitty.plan command)
├── contracts/           # Phase 1 output (/spec-kitty.plan command)
└── tasks.md             # Phase 2 output (/spec-kitty.tasks command - NOT created by /spec-kitty.plan)
```

### Source Code (repository root)

```
backend/
├── src/
│   ├── models/
│   ├── services/
│   ├── api/
│   ├── middleware/
│   ├── utils/
│   └── config/
├── tests/
│   ├── unit/
│   ├── integration/
│   └── contract/
├── package.json
├── tsconfig.json
└── Dockerfile

frontend/
├── src/
│   ├── components/
│   ├── pages/
│   ├── services/
│   ├── hooks/
│   ├── contexts/
│   ├── types/
│   └── utils/
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── package.json
├── tsconfig.json
├── vite.config.ts
└── Dockerfile

docker-compose.yml
docker-compose.prod.yml
k8s/
├── base/
├── overlays/
│   ├── dev/
│   └── prod/
└── README.md
```

**Structure Decision**: Web application (Option 2) — React + TypeScript frontend (Vite) and Node.js + TypeScript backend (NestJS) as separate projects under `frontend/` and `backend/` directories. Docker Compose for local development under `docker-compose.yml`; Kubernetes manifests under `k8s/` with base and overlay structure for dev/prod environments.

## Complexity Tracking

*Fill ONLY if Charter Check has violations that must be justified*

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| [e.g., 4th project] | [current need] | [why 3 projects insufficient] |
| [e.g., Repository pattern] | [specific problem] | [why direct DB access insufficient] |

## Implementation Concern Map

*Include this section when the mission has multiple distinct architectural areas that inform how tasks are decomposed.*

> **Note**: Implementation concerns are NOT work packages and are NOT executable units.
> `/spec-kitty.tasks` translates these into executable WPs — one concern may become
> multiple WPs; multiple small concerns may merge into one WP. Do not label concerns
> with WP-style IDs or sequencing language.

### IC-01 — Authentication & Authorization Foundation

- **Purpose**: Core authentication (JWT + refresh tokens) and RBAC for all user roles (Customer, Support Agent, Support Supervisor, Administrator) — the security boundary for the entire platform
- **Relevant requirements**: FR-001, FR-002, FR-003, FR-004, FR-005, FR-006, C-009
- **Affected surfaces**: backend/src/models/auth.*, backend/src/services/auth.*, backend/src/middleware/auth.*, backend/src/api/auth.*, frontend/src/contexts/AuthContext.*, frontend/src/services/auth.*
- **Sequencing/depends-on**: none (foundational — must be first)
- **Risks**: Token refresh race conditions, role escalation bugs, session management across SSE connections

### IC-02 — Customer Management

- **Purpose**: Customer CRUD, search, and case history — supports both customer self-service portal and agent-facing customer lookup
- **Relevant requirements**: FR-010, FR-011, FR-012, FR-013, FR-014
- **Affected surfaces**: backend/src/models/customer.*, backend/src/services/customer.*, backend/src/api/customers.*, frontend/src/pages/Customer*.*, frontend/src/services/customers.*
- **Sequencing/depends-on**: IC-01 (requires auth)
- **Risks**: PII handling, search performance at scale, organization relationship integrity

### IC-03 — Case Lifecycle Engine

- **Purpose**: Core case state machine (New → Open → Pending Customer/Pending Internal → Resolved → Closed) with SLA timer pause/resume logic, routing rules, catch-all queue fallback, and email-to-case ingestion
- **Relevant requirements**: FR-020, FR-021, FR-022, FR-023, FR-024, FR-025, FR-026, C-014
- **Affected surfaces**: backend/src/models/case.*, backend/src/services/case.*, backend/src/services/routing.*, backend/src/services/email-ingestion.*, backend/src/api/cases.*, frontend/src/pages/Case*.*, frontend/src/services/cases.*
- **Sequencing/depends-on**: IC-01, IC-02
- **Risks**: State transition bugs, SLA timer accuracy during pause/resume, routing rule evaluation failures, email parsing edge cases, catch-all queue alert reliability

### IC-04 — Agent Case Management & Operations

- **Purpose**: Agent-facing case operations — assignment, status changes, priority/category updates, escalation, collision prevention (real-time drafting lock), bulk actions, and single-action response + Pending Customer transition
- **Relevant requirements**: FR-030, FR-031, FR-032, FR-033, FR-034, FR-035, FR-036, FR-037, FR-038, FR-039, FR-040, FR-041
- **Affected surfaces**: backend/src/services/case-operations.*, backend/src/services/collision.*, backend/src/api/cases.*, frontend/src/components/CaseEditor.*, frontend/src/hooks/useCaseDraft.*
- **Sequencing/depends-on**: IC-03
- **Risks**: Real-time collision prevention race conditions, bulk action authorization consistency, optimistic locking conflicts, escalation audit trail completeness

### IC-05 — Communications & Macros

- **Purpose**: Customer-visible messages vs internal notes, chronological display, attachments, macros (templates + field updates), keyboard shortcuts for triage actions
- **Relevant requirements**: FR-045, FR-046, FR-047, FR-048, FR-049, FR-050, FR-051, FR-052, FR-053
- **Affected surfaces**: backend/src/models/communication.*, backend/src/services/communication.*, backend/src/services/macro.*, backend/src/api/communications.*, frontend/src/components/MessageThread.*, frontend/src/components/MacroPicker.*
- **Sequencing/depends-on**: IC-03
- **Risks**: Attachment storage integration (S3/MinIO), macro field-update side effects, keyboard shortcut conflicts, message ordering with real-time updates

### IC-06 — Search, Filtering & Queue Views

- **Purpose**: Full-text case search, multi-dimensional filtering, queue views with sorting/pagination, supervisor views (unassigned/overdue), bulk actions from queue
- **Relevant requirements**: FR-060, FR-061, FR-062, FR-063, FR-064, FR-065, FR-066
- **Affected surfaces**: backend/src/services/search.*, backend/src/api/search.*, backend/src/api/queues.*, frontend/src/pages/QueueView.*, frontend/src/components/CaseFilters.*, frontend/src/hooks/useCaseSearch.*
- **Sequencing/depends-on**: IC-03
- **Risks**: PostgreSQL full-text search performance, filter combination complexity, bulk action atomicity, pagination consistency with real-time updates

### IC-07 — SLA Management

- **Purpose**: Configurable response/resolution targets by category/priority/queue, target timestamp calculation, automatic pause/resume in Pending Customer, risk/breach identification, supervisor views, escalation event recording
- **Relevant requirements**: FR-070, FR-071, FR-072, FR-073, FR-074, FR-075
- **Affected surfaces**: backend/src/models/sla.*, backend/src/services/sla.*, backend/src/jobs/sla-monitor.*, backend/src/api/sla.*, frontend/src/pages/SLADashboard.*
- **Sequencing/depends-on**: IC-03
- **Risks**: Timer accuracy across restarts, timezone handling, breach detection latency, configuration change impact on in-flight cases

### IC-08 — Dashboards & Reporting

- **Purpose**: Agent dashboard (assigned/open/attention cases), supervisor dashboard (volume, workload, backlog, aging, overdue), volume reporting by dimensions, average response/resolution metrics, drill-down to case lists, authorization-scoped reports
- **Relevant requirements**: FR-080, FR-081, FR-082, FR-083, FR-084, FR-085
- **Affected surfaces**: backend/src/services/reporting.*, backend/src/api/reports.*, frontend/src/pages/AgentDashboard.*, frontend/src/pages/SupervisorDashboard.*, frontend/src/components/ReportDrilldown.*
- **Sequencing/depends-on**: IC-03, IC-06, IC-07
- **Risks**: Query performance on large datasets, real-time dashboard updates, authorization scoping on reports, metric calculation accuracy

### IC-09 — Administration & Configuration

- **Purpose**: Manage statuses, priorities, categories, queues, permissions, service-level rules; prevent deletion of referenced configs; immutable catch-all queue; audit event review
- **Relevant requirements**: FR-090, FR-091, FR-092, FR-093, FR-094, FR-095, FR-096
- **Affected surfaces**: backend/src/models/config.*, backend/src/services/config.*, backend/src/api/admin/*.config.*, frontend/src/pages/Admin*.*, frontend/src/services/admin.*
- **Sequencing/depends-on**: IC-01
- **Risks**: Referential integrity enforcement on config deletion, catch-all queue immutability, permission matrix complexity, configuration change propagation

### IC-10 — Audit Logging

- **Purpose**: Immutable audit trail for case changes, assignments, status changes, escalations, admin config changes; event type, actor, timestamp, before/after values; tamper-proof for normal users
- **Relevant requirements**: FR-100, FR-101, FR-102, FR-103
- **Affected surfaces**: backend/src/models/audit.*, backend/src/services/audit.*, backend/src/middleware/audit.*, backend/src/api/audit.*
- **Sequencing/depends-on**: IC-01 (cross-cutting — integrates with all case/admin operations)
- **Risks**: Performance impact of audit writes, storage growth, tamper-proof guarantees, query performance for audit review

### IC-11 — Real-Time Event Distribution

- **Purpose**: Server→client push via SSE for case/queue updates, notifications, collision warnings; graceful degradation if real-time unavailable; integration with all mutation operations
- **Relevant requirements**: FR-110, FR-111, NFR-011
- **Affected surfaces**: backend/src/services/realtime.*, backend/src/api/sse.*, frontend/src/hooks/useRealtime.*, frontend/src/contexts/RealtimeContext.*
- **Sequencing/depends-on**: IC-01, IC-03 (cross-cutting — integrates with all mutation operations)
- **Risks**: Connection management at scale, event ordering guarantees, reconnection logic, fallback to polling, memory leaks

### IC-12 — Email Integration (Provider-Independent)

- **Purpose**: Webhook-based email-to-case ingestion (Postmark) with provider-independent boundary, inbound email association with existing cases, outbound notification delivery
- **Relevant requirements**: FR-025, FR-026, C-014
- **Affected surfaces**: backend/src/services/email-ingestion.*, backend/src/services/email-outbound.*, backend/src/api/webhooks/email.*, backend/src/adapters/email/PostmarkAdapter.*
- **Sequencing/depends-on**: IC-03
- **Risks**: Webhook signature verification, email parsing reliability, provider API changes, bounce/complaint handling, rate limiting

### IC-13 — Async Processing & Background Jobs

- **Purpose**: Bull/BullMQ (Redis-based) for SLA monitoring, email delivery, report generation, audit writes, and other deferred work — retries, scheduling, priority, metrics, horizontal scaling
- **Relevant requirements**: C-007, FR-073, FR-100
- **Affected surfaces**: backend/src/jobs/*, backend/src/config/queue.*, backend/src/services/queue.*
- **Sequencing/depends-on**: IC-01 (foundational infrastructure)
- **Risks**: Job idempotency, Redis connection resilience, queue backlog monitoring, priority inversion, dead letter handling

### IC-14 — File Attachment Handling

- **Purpose**: Upload/download of case attachments via S3-compatible storage (MinIO locally, configurable S3 endpoint for prod), size/type validation, virus scanning hook, signed URLs for secure access
- **Relevant requirements**: FR-049, C-014 (attachment storage assumption)
- **Affected surfaces**: backend/src/services/attachment.*, backend/src/api/attachments.*, frontend/src/components/AttachmentUploader.*, frontend/src/services/attachments.*
- **Sequencing/depends-on**: IC-01, IC-12 (storage config)
- **Risks**: Signed URL expiration, multipart upload for large files, MinIO/S3 API compatibility, storage cost control

### IC-15 — Observability & Operational Tooling

- **Purpose**: Structured logging (Pino), metrics (Prometheus), distributed tracing (OpenTelemetry), health checks, Docker/K8s readiness/liveness probes, seed/demo data (NFR-008)
- **Relevant requirements**: NFR-005, NFR-006, NFR-008, C-011, C-012, C-013
- **Affected surfaces**: backend/src/config/observability.*, backend/src/middleware/logging.*, backend/src/middleware/metrics.*, docker-compose.yml, k8s/, backend/prisma/seed.*
- **Sequencing/depends-on**: IC-01 (foundational infrastructure)
- **Risks**: Log volume/cost, metric cardinality explosion, trace sampling configuration, demo data realism vs. privacy
