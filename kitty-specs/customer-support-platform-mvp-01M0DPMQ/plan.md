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
**Primary Dependencies**: React 18+, Express/Fastify, PostgreSQL client (pg), Redis client (ioredis), JWT (jsonwebtoken), Zod (validation)
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
**Email Provider**: [NEEDS CLARIFICATION: SendGrid, Mailgun, Postmark, or other webhook-based provider] <!-- decision_id: TBD -->
**CI/CD Platform**: [NEEDS CLARIFICATION: GitHub Actions, GitLab CI, or other] <!-- decision_id: TBD -->
**Observability Stack**: [NEEDS CLARIFICATION: Structured logging (Pino/Winston) + metrics (Prometheus) + tracing (OpenTelemetry)] <!-- decision_id: TBD -->
**Attachment Storage**: [NEEDS CLARIFICATION: Local filesystem, S3-compatible, or database BLOB] <!-- decision_id: TBD -->
**Container Orchestration**: [NEEDS CLARIFICATION: Docker Compose (demo) vs Kubernetes] <!-- decision_id: TBD -->

## Charter Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

[Gates determined based on charter file]

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
<!--
  ACTION REQUIRED: Replace the placeholder tree below with the concrete layout
  for this mission. Delete unused options and expand the chosen structure with
  real paths (e.g., apps/admin, packages/something). The delivered plan must
  not include Option labels.
-->

```
# [REMOVE IF UNUSED] Option 1: Single project (DEFAULT)
src/
├── models/
├── services/
├── cli/
└── lib/

tests/
├── contract/
├── integration/
└── unit/

# [REMOVE IF UNUSED] Option 2: Web application (when "frontend" + "backend" detected)
backend/
├── src/
│   ├── models/
│   ├── services/
│   └── api/
└── tests/

frontend/
├── src/
│   ├── components/
│   ├── pages/
│   └── services/
└── tests/

# [REMOVE IF UNUSED] Option 3: Mobile + API (when "iOS/Android" detected)
api/
└── [same as backend above]

ios/ or android/
└── [platform-specific structure: feature modules, UI flows, platform tests]
```

**Structure Decision**: [Document the selected structure and reference the real
directories captured above]

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

### IC-01 — [Name]

- **Purpose**: [One sentence: what this concern addresses and why it matters]
- **Relevant requirements**: [FR-### refs from spec.md]
- **Affected surfaces**: [File paths or module names this concern touches]
- **Sequencing/depends-on**: [IC-## IDs this concern must follow, or "none"]
- **Risks**: [Key coordination notes or implementation risks]

### IC-02 — [Name]

- **Purpose**: [One sentence]
- **Relevant requirements**: [FR-### refs]
- **Affected surfaces**: [Paths/modules]
- **Sequencing/depends-on**: [IC-## or "none"]
- **Risks**: [Notes]
