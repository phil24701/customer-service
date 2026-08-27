---
schema_version: 1
artifact_type: spec-kitty.analysis-report
command: /spec-kitty.analyze
mission_slug: customer-support-platform-mvp-01M0DPMQ
mission_id: 01M0DPMQRGYVXD960W4S34P2NH
generated_at: '2026-08-26T00:50:58.553611+00:00'
analyzer_agent: unknown
input_artifacts:
  spec.md:
    path: /home/sweeney/Documents/projects/customer-service/kitty-specs/customer-support-platform-mvp-01M0DPMQ/spec.md
    sha256: f076b23df7d50343bd6d50e8199c6194a9fdcd0bebf166ea0ec76a290a997c39
  plan.md:
    path: /home/sweeney/Documents/projects/customer-service/kitty-specs/customer-support-platform-mvp-01M0DPMQ/plan.md
    sha256: 0d0ffca9533871987ad04753f4a6b4d7c5c4700fa4fd1612f6083c2b33026528
  tasks.md:
    path: /home/sweeney/Documents/projects/customer-service/kitty-specs/customer-support-platform-mvp-01M0DPMQ/tasks.md
    sha256: 2b2708526be4ab0af69f6c003ba85017ce3e0dd83903215023ce8bf5449c73c0
  charter:
    path:
    sha256:
verdict: blocked
issue_counts:
  low: 18
  critical: 0
  medium: 18
  high: 14
  info: 0
findings:
- id: A1
  severity: high
  category: ambiguity
  summary: NFR-010 'adapt reasonably to smaller screens' lacks measurable criteria (no breakpoint definitions, no specific device targets)
- id: A2
  severity: high
  category: ambiguity
  summary: NFR-005 'practical health/diagnostic monitoring' undefined - no specific metrics, alerting thresholds, or SLOs specified
- id: A3
  severity: high
  category: ambiguity
  summary: C-006 'search implementation to be determined in technical design' - decision deferred but tasks assume PostgreSQL full-text (T033)
- id: A4
  severity: high
  category: ambiguity
  summary: C-007 'async mechanism to be selected during technical design' - decision deferred but tasks assume BullMQ (T051-T053)
- id: A5
  severity: high
  category: ambiguity
  summary: C-008 'real-time mechanism to be selected during technical design' - decision deferred but tasks assume SSE (T054-T056)
- id: A6
  severity: high
  category: ambiguity
  summary: C-012 'CI/CD platform details to be selected' - decision deferred but tasks assume GitHub Actions (T069)
- id: A7
  severity: high
  category: ambiguity
  summary: C-013 'observability tooling to be selected' - decision deferred but tasks assume Pino+Prometheus+OpenTelemetry (T004, T005)
- id: A8
  severity: high
  category: ambiguity
  summary: C-014 'email provider selected during technical design' - decision deferred but tasks assume Postmark (T047-T050)
- id: A9
  severity: high
  category: ambiguity
  summary: "Assumption: 'Attachment storage strategy to be determined in technical design' but tasks assume S3-compatible/MinIO (T025)"
- id: A10
  severity: high
  category: ambiguity
  summary: "Assumption: 'Container orchestration to be determined' but tasks assume Docker Compose + K8s (T003, T070)"
- id: A11
  severity: high
  category: ambiguity
  summary: "Assumption: 'Search implementation to be determined' but tasks assume PostgreSQL full-text (T033)"
- id: A12
  severity: high
  category: inconsistency
  summary: Plan.md Technical Context lists decisions (NestJS, Prisma, BullMQ, SSE, Postmark, GitHub Actions, etc.) that spec.md Section 6 says are 'deferred to technical design with ADR'
- id: A46
  severity: high
  category: coverage
  summary: FR-103 'Normal users shall not be able to alter or delete audit history' - no task for audit immutability enforcement (DB constraints, API protection)
- id: A47
  severity: high
  category: coverage
  summary: FR-095 'immutable catch-all queue cannot be deactivated or deleted' - no task for DB-level constraint (CHECK constraint or trigger)
- id: A13
  severity: medium
  category: inconsistency
  summary: Plan.md IC-03 mentions 'email-to-case ingestion' but IC-12 is separate Email Integration concern - overlapping scope
- id: A14
  severity: medium
  category: inconsistency
  summary: Plan.md IC-01 includes 'session management across SSE connections' but SSE is in IC-11 - cross-cutting concern not clearly managed
- id: A15
  severity: medium
  category: inconsistency
  summary: Plan.md IC-10 (Audit Logging) depends on IC-01 but audit is cross-cutting - should integrate with all ICs not just IC-01
- id: A16
  severity: medium
  category: inconsistency
  summary: Plan.md IC-14 (File Attachment Handling) depends on IC-12 (storage config) but IC-12 is email integration - wrong dependency
- id: A17
  severity: medium
  category: underspecification
  summary: FR-039 collision prevention requires 'real-time warning' and 'editor temporarily locked' but no specification of lock duration, timeout behavior, or conflict resolution when lock expires
- id: A18
  severity: medium
  category: underspecification
  summary: FR-026 'associate inbound email when valid case relationship can be determined' - no definition of what constitutes valid relationship (subject prefix, Message-ID, References header?)
- id: A19
  severity: medium
  category: underspecification
  summary: FR-072 SLA pause/resume - no specification of behavior when case moves Pending Customer → Open → Pending Customer multiple times (cumulative pause tracking)
- id: A20
  severity: medium
  category: underspecification
  summary: FR-094 'prevent deactivation until reassigned' - no specification of how reassignment is validated or what 'made valid' means
- id: A21
  severity: medium
  category: underspecification
  summary: FR-111 'real-time updates as defined by technical design' - circular reference, no concrete event list in spec
- id: A22
  severity: medium
  category: underspecification
  summary: NFR-011 graceful degradation - no specification of fallback behavior (polling interval, optimistic locking conflict resolution)
- id: A34
  severity: medium
  category: inconsistency
  summary: Tasks.md T032 (bulk actions) includes 'queue_change' but FR-065 only mentions 'bulk assignment and bulk status changes'
- id: A35
  severity: medium
  category: inconsistency
  summary: Plan.md IC-09 mentions 'permissions' management but no FR for permission matrix - FR-091 only says 'configure applicable permissions'
- id: A41
  severity: medium
  category: underspecification
  summary: FR-023 catch-all queue + admin alert - no specification of alert delivery mechanism (email, in-app, webhook, PagerDuty?)
- id: A42
  severity: medium
  category: underspecification
  summary: FR-041 Pending Internal requires 'identification of responsible internal department or owner' - no spec for department vs owner distinction or validation
- id: A43
  severity: medium
  category: inconsistency
  summary: Tasks.md T017 creates 'Case, Status, Priority, Category, Queue, SLARule, BusinessHours models' but plan.md IC-03 lists only 'Case, SLA' models
- id: A44
  severity: medium
  category: inconsistency
  summary: Tasks.md T023 includes 'EmailIngestionLog' model but plan.md IC-05 (Communications) doesn't list it; IC-12 (Email) does
- id: A45
  severity: medium
  category: inconsistency
  summary: Plan.md IC-06 (Search) and IC-07 (SLA) both depend on IC-03 - correct, but tasks split SLA across WP04 (T021) and WP08 (T036,T037)
- id: A48
  severity: medium
  category: inconsistency
  summary: Tasks.md T019 'routing engine with catch-all queue fallback & admin alert' but no task for admin alert delivery mechanism (email, in-app, webhook?)
- id: A23
  severity: low
  category: duplication
  summary: FR-034 and FR-035 both reference case statuses and activity history - overlapping but not identical
- id: A24
  severity: low
  category: duplication
  summary: Plan.md IC-03 and IC-04 both cover case operations - IC-03 is lifecycle engine, IC-04 is agent operations, but boundary unclear
- id: A25
  severity: low
  category: duplication
  summary: Tasks T021 (SLA calculation) in WP04 and T036 (SLA config) in WP08 - SLA concerns split across WPs
- id: A26
  severity: low
  category: coverage
  summary: NFR-001 (layered architecture) - no explicit task for architecture enforcement or code structure validation
- id: A27
  severity: low
  category: coverage
  summary: NFR-002 (server-side validation/auth) - covered by WP02 auth but no task for API validation middleware
- id: A28
  severity: low
  category: coverage
  summary: NFR-003 (referential integrity/indexes) - partially covered by Prisma schema but no task for index validation or migration review
- id: A29
  severity: low
  category: coverage
  summary: NFR-004 (clear validation/error handling) - no explicit task for error handling standards or validation patterns
- id: A30
  severity: low
  category: coverage
  summary: NFR-006 (reproducible deploy) - covered by T069/T070 but no task for environment parity validation
- id: A31
  severity: low
  category: coverage
  summary: NFR-007 (externalized secrets) - no task for secret management setup (Vault, .env, Kubernetes secrets)
- id: A32
  severity: low
  category: coverage
  summary: NFR-009 (documented RESTful APIs) - OpenAPI spec exists but no task for API documentation generation/publishing
- id: A33
  severity: low
  category: coverage
  summary: C-016 (multi-tenant auth) - no task for tenant selection UI or credential validation before tenant disclosure
- id: A36
  severity: low
  category: ambiguity
  summary: Plan.md IC-01 'session management across SSE connections' - unclear how JWT refresh works with SSE (separate connection, token in query param?)
- id: A37
  severity: low
  category: ambiguity
  summary: "Plan.md 'Performance Goals: support 10k concurrent users' - no load testing task or capacity planning validation"
- id: A38
  severity: low
  category: coverage
  summary: No task for C-005 Redis caching strategy (what gets cached, TTL, invalidation)
- id: A39
  severity: low
  category: ambiguity
  summary: FR-053 keyboard shortcuts - no specification of which shortcuts, conflicts with browser/OS shortcuts, accessibility
- id: A40
  severity: low
  category: coverage
  summary: No task for C-015 AI exclusion verification (ensure no AI code accidentally included)
- id: A49
  severity: low
  category: ambiguity
  summary: Plan.md IC-11 'graceful degradation if real-time unavailable' but tasks assume SSE - no task for WebSocket fallback or polling implementation
- id: A50
  severity: low
  category: ambiguity
  summary: "Success Criteria: 'Agents are protected against simultaneous response editing' - no measurable criteria (lock acquisition time, conflict rate)"
---


