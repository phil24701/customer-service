# Work Packages: Customer Support Platform MVP

**Mission**: customer-support-platform-mvp-01M0DPMQ  
**Branch**: epic1  
**Generated**: 2026-08-25T18:45:19Z  

---

## Subtask Index

| ID | Description | WP | Parallel |
|----|-------------|----|----------|
| T001 | Initialize NestJS backend project structure with TypeScript, ESLint, Prettier | WP01 |  |
| T002 | Configure Prisma ORM with PostgreSQL schema from data-model.md | WP01 |  |
| T003 | Set up Docker Compose for local development (PostgreSQL, Redis, MinIO) | WP01 | [P] |
| T004 | Configure Pino logging, Prometheus metrics, OpenTelemetry tracing | WP01 | [P] |
| T005 | Create base NestJS modules: AppModule, ConfigModule, DatabaseModule | WP01 |  |
| T005b | Create AuditLog Prisma model with immutable fields (created_at only, no updated_at), add DB trigger to prevent UPDATE/DELETE | WP01 |  |
| T006 | Implement JWT authentication with access/refresh tokens (Argon2id) | WP02 |  |
| T007 | Build login, refresh, logout, me endpoints per OpenAPI contracts | WP02 |  |
| T008 | Create RBAC middleware for 4 roles: customer, agent, supervisor, admin | WP02 |  |
| T009 | Implement session management with refresh token rotation/revocation | WP02 |  |
| T010 | Add rate limiting on auth endpoints (5 attempts/15min per IP) | WP02 | [P] |
| T011 | Write unit/integration tests for auth flow | WP02 | [P] |
| T011a | Design audit immutability: API protection (no PATCH/DELETE endpoints for audit), RLS policies | WP02 |  |
| T012 | Create User, UserRole, Session, Tenant Prisma models and migrations | WP03 |  |
| T013 | Build user management CRUD endpoints (admin only) | WP03 |  |
| T014 | Create Customer, Organization models and customer search/list endpoints | WP03 | [P] |
| T015 | Implement customer profile with case history view | WP03 |  |
| T016 | Seed demo data: tenant, admin, agents, supervisor, customers, org | WP03 | [P] |
| T017 | Create Case, Status, Priority, Category, Queue, SLARule, BusinessHours models | WP04 |  |
| T018 | Implement case state machine with transition guards (6 statuses) | WP04 |  |
| T019 | Build case routing engine with catch-all queue fallback & admin alert | WP04 |  |
| T020 | Create case CRUD endpoints with validation per OpenAPI | WP04 |  |
| T021 | Implement SLA target calculation on case creation (business hours) | WP04 | [P] |
| T022 | Add case activity log (denormalized) and full audit trail | WP04 | [P] |
| T023 | Create Communication, Attachment, Macro, AuditLog, EmailIngestionLog models | WP05 |  |
| T024 | Build communications endpoints (reply, internal note, chronological list) | WP05 |  |
| T025 | Implement attachment upload via S3/MinIO signed URLs | WP05 | [P] |
| T026 | Create macro system: templates with field updates, keyboard shortcuts | WP05 |  |
| T027 | Add collision prevention: drafting lock acquire/release via SSE | WP05 |  |
| T028 | Implement case assignment/reassignment with supervisor override | WP06 |  |
| T029 | Build status transitions with auto Pending Customer → Open on inbound | WP06 |  |
| T030 | Implement Pending Internal requiring internal owner/department | WP06 |  |
| T031 | Add escalation as distinct operational event with audit | WP06 | [P] |
| T032 | Create bulk actions: assign, status, priority, category, queue (≤100 cases) | WP06 |  |
| T033 | Build search with PostgreSQL full-text (tsvector/GIN) and multi-filter | WP07 |  |
| T034 | Create queue views with case counts, sorting, pagination | WP07 | [P] |
| T035 | Implement supervisor views: unassigned, overdue, SLA risk cases | WP07 | [P] |
| T036 | Add SLA configuration: rules by category/priority/queue with business hours | WP08 |  |
| T037 | Implement SLA monitor background job (BullMQ) for breach detection | WP08 |  |
| T038 | Build agent dashboard: assigned/open/attention cases, queue counts | WP09 |  |
| T039 | Build supervisor dashboard: volume, workload, backlog, aging, overdue | WP09 | [P] |
| T040 | Implement volume reporting by status/priority/category/time period | WP09 | [P] |
| T041 | Add metrics: avg response/resolution, SLA compliance, drill-down to cases | WP09 | [P] |
| T042 | Create admin config endpoints: statuses, priorities, categories, queues | WP10 |  |
| T043 | Implement SLA rule management CRUD | WP10 | [P] |
| T044 | Add macro management CRUD with shortcut keys | WP10 | [P] |
| T045 | Enforce referential integrity: prevent deletion of referenced config | WP10 |  |
| T046 | Make catch-all queue immutable (cannot deactivate/delete) | WP10 | [P] |
| T047 | Build email ingestion webhook endpoint (Postmark adapter) | WP11 |  |
| T048 | Implement inbound email parsing & case association (threading) | WP11 |  |
| T049 | Add webhook signature verification and dead-letter queue for failures | WP11 | [P] |
| T050 | Create outbound email notification service (provider-independent) | WP11 | [P] |
| T051 | Set up BullMQ queues: sla-monitor, email-delivery, audit-write, reports | WP12 |  |
| T052 | Configure job retries, exponential backoff, dead letter queues | WP12 | [P] |
| T053 | Add priority queues for SLA breach alerts (high) vs reports (low) | WP12 | [P] |
| T054 | Implement graceful SSE degradation → optimistic locking fallback | WP13 |  |
| T055 | Build SSE connection management with reconnection logic | WP13 |  |
| T056 | Create real-time events: case.updated, queue.changed, collision.warning | WP13 |  |
| T057 | Initialize React + TypeScript + Vite frontend project | WP14 |  |
| T058 | Configure React Query, React Router, React Hook Form, Zod schemas | WP14 | [P] |
| T059 | Set up atomic design component structure (atoms/molecules/organisms/pages) | WP14 | [P] |
| T060 | Create AuthContext, useAuth hook, login page, protected routes | WP15 |  |
| T061 | Build customer portal: case list, case detail, create case, communications | WP15 |  |
| T062 | Build agent workspace: queue views, case editor, collision warning UI | WP16 |  |
| T063 | Implement macro picker, keyboard shortcuts, single-action reply+pending | WP16 | [P] |
| T064 | Build supervisor dashboard: workload, queue backlog, aging, SLA risk | WP17 |  |
| T065 | Create admin configuration UI: statuses, priorities, categories, queues | WP17 | [P] |
| T066 | Add SLA rule builder, macro manager, audit log viewer | WP17 | [P] |
| T067 | Implement real-time hooks: useRealtime, useCaseDraft, EventSource wrapper | WP18 |  |
| T068 | Add SSE integration to all mutation operations for live updates | WP18 | [P] |
| T069 | Configure GitHub Actions CI/CD: lint, typecheck, test, build, deploy | WP19 |  |
| T070 | Create Kubernetes manifests (base + dev/prod overlays with kustomize) | WP19 | [P] |
| T071 | Write E2E tests for critical user flows (Playwright) | WP19 | [P] |
| T072 | Verify seed data works end-to-end for portfolio demo | WP19 | [P] |

---

## Work Packages

### WP01 — Foundation: Backend Project Setup & Infrastructure

**Goal**: Initialize NestJS backend with all foundational infrastructure  
**Priority**: Critical (blocks all other backend work)  
**Independent Test**: `npm run build && npm run test` passes; Docker Compose starts all services  
**Subtasks**: T001, T002, T003, T004, T005, T005b  
**Dependencies**: None  
**Parallel Opportunities**: T003, T004 can run in parallel after T001  
**Risks**: Prisma schema must match data-model.md exactly; Docker Compose networking  
**Estimated Prompt Size**: ~350 lines  

---

### WP02 — Authentication & Authorization Foundation (Epic 1)

**Goal**: Complete JWT + RBAC authentication system  
**Priority**: Critical (Epic 1 deliverable)  
**Independent Test**: Login → access protected endpoint → refresh token → logout works; RBAC denies unauthorized roles  
**Subtasks**: T006, T007, T008, T009, T010, T011, T011a  
**Dependencies**: WP01  
**Parallel Opportunities**: T010, T011 can run in parallel after T006-T009  
**Risks**: Token refresh race conditions; refresh token rotation edge cases; rate limiting integration  
**Estimated Prompt Size**: ~450 lines  

---

### WP03 — User & Customer Management

**Goal**: User administration and customer management with demo seed data  
**Priority**: High (Epic 1 completion)  
**Independent Test**: Admin can CRUD users/roles; customer search works; demo data seeds correctly  
**Subtasks**: T012, T013, T014, T015, T016  
**Dependencies**: WP02  
**Parallel Opportunities**: T014, T016 can run in parallel  
**Risks**: PII handling in customer data; search performance at scale; organization relationship integrity  
**Estimated Prompt Size**: ~380 lines  

---

### WP04 — Case Lifecycle Engine (Core)

**Goal**: Core case state machine, routing, SLA calculation, audit trail  
**Priority**: Critical (heart of the platform)  
**Independent Test**: Case transitions through all 6 statuses correctly; routing falls back to catch-all; SLA targets calculated; audit logs generated  
**Subtasks**: T017, T018, T019, T020, T021, T022  
**Dependencies**: WP03  
**Parallel Opportunities**: T021, T022 can run in parallel after T017-T020  
**Risks**: State transition bugs; SLA timer accuracy during pause/resume; routing rule evaluation failures; email parsing edge cases; catch-all queue alert reliability  
**Estimated Prompt Size**: ~550 lines  

---

### WP05 — Communications, Attachments, Macros & Collision Prevention

**Goal**: Full communication system with macros and real-time collision prevention  
**Priority**: High  
**Independent Test**: Customer-visible vs internal notes distinguished; attachments upload/download via signed URLs; macros populate text + update fields; collision lock acquires/releases correctly  
**Subtasks**: T023, T024, T025, T026, T027  
**Dependencies**: WP04  
**Parallel Opportunities**: T025 can run in parallel after T023  
**Risks**: Attachment storage integration (S3/MinIO); macro field-update side effects; keyboard shortcut conflicts; message ordering with real-time updates  
**Estimated Prompt Size**: ~450 lines  

---

### WP06 — Agent Case Operations & Bulk Actions

**Goal**: Agent-facing case operations including bulk actions and escalation  
**Priority**: High  
**Independent Test**: Assignment/reassignment works; status transitions enforce guards; Pending Internal requires owner/dept; bulk actions respect auth rules; escalation recorded  
**Subtasks**: T028, T029, T030, T031, T032  
**Dependencies**: WP04, WP05  
**Parallel Opportunities**: T031 can run in parallel  
**Risks**: Real-time collision prevention race conditions; bulk action authorization consistency; optimistic locking conflicts; escalation audit trail completeness  
**Estimated Prompt Size**: ~420 lines  

---

### WP07 — Search, Filtering & Queue Views

**Goal**: Full-text search, multi-dimensional filtering, queue and supervisor views  
**Priority**: High  
**Independent Test**: Search returns ranked results; filters combine correctly; queue views paginate; supervisor sees unassigned/overdue/SLA-risk  
**Subtasks**: T033, T034, T035  
**Dependencies**: WP04  
**Parallel Opportunities**: T034, T035 can run in parallel  
**Risks**: PostgreSQL full-text search performance; filter combination complexity; bulk action atomicity; pagination consistency with real-time updates  
**Estimated Prompt Size**: ~350 lines  

---

### WP08 — SLA Management & Monitoring

**Goal**: Configurable SLA rules with background monitoring and breach detection  
**Priority**: High  
**Independent Test**: SLA rules by category/priority/queue calculate targets; monitor job detects approaching/breached cases; escalation events recorded  
**Subtasks**: T036, T037  
**Dependencies**: WP04, WP12 (BullMQ setup)  
**Parallel Opportunities**: T037 can run after T036 + WP12  
**Risks**: Timer accuracy across restarts; timezone handling; breach detection latency; config change impact on in-flight cases  
**Estimated Prompt Size**: ~300 lines  

---

### WP09 — Dashboards & Reporting

**Goal**: Agent/supervisor dashboards with volume reporting and metrics  
**Priority**: High  
**Independent Test**: Agent dashboard shows assigned/open/attention; supervisor dashboard shows volume/workload/backlog/aging/overdue; reports drill down to cases; auth scope respected  
**Subtasks**: T038, T039, T040, T041  
**Dependencies**: WP04, WP07, WP08  
**Parallel Opportunities**: T039, T040, T041 can run in parallel after T038  
**Risks**: Query performance on large datasets; real-time dashboard updates; authorization scoping on reports; metric calculation accuracy  
**Estimated Prompt Size**: ~450 lines  

---

### WP10 — Administration & Configuration

**Goal**: Full admin configuration with referential integrity enforcement  
**Priority**: High  
**Independent Test**: Admin can CRUD statuses/priorities/categories/queues/SLA rules/macros; referenced config cannot be deleted; catch-all queue immutable; deactivation blocked until reassignment  
**Subtasks**: T042, T043, T044, T045, T046  
**Dependencies**: WP04  
**Parallel Opportunities**: T043, T044, T046 can run in parallel after T042  
**Risks**: Referential integrity enforcement on config deletion; catch-all queue immutability; permission matrix complexity; configuration change propagation  
**Estimated Prompt Size**: ~400 lines  

---

### WP11 — Email Integration (Postmark)

**Goal**: Webhook-based email-to-case ingestion with provider-independent boundary  
**Priority**: High  
**Independent Test**: Postmark webhook creates case; inbound replies attach to existing case; signature verified; failed ingestion goes to dead-letter; outbound notifications send  
**Subtasks**: T047, T048, T049, T050  
**Dependencies**: WP04, WP05  
**Parallel Opportunities**: T049, T050 can run in parallel after T047-T048  
**Risks**: Webhook signature verification; email parsing reliability; provider API changes; bounce/complaint handling; rate limiting  
**Estimated Prompt Size**: ~400 lines  

---

### WP12 — Async Processing & Background Jobs (BullMQ)

**Goal**: Redis-based job queues for all deferred work  
**Priority**: High (enables WP08, WP11, WP04 audit)  
**Independent Test**: Jobs process with retries/backoff; dead letters captured; priority queues work; metrics exposed to Prometheus  
**Subtasks**: T051, T052, T053  
**Dependencies**: WP01  
**Parallel Opportunities**: T052, T053 can run in parallel after T051  
**Risks**: Job idempotency; Redis connection resilience; queue backlog monitoring; priority inversion; dead letter handling  
**Estimated Prompt Size**: ~300 lines  

---

### WP13 — Real-Time Event Distribution (SSE)

**Goal**: Server→client push via SSE with graceful degradation  
**Priority**: High (enables collision prevention, live updates)  
**Independent Test**: SSE connects/reconnects; events delivered <100ms p95; collision warnings work; graceful fallback to optimistic locking when SSE unavailable  
**Subtasks**: T054, T055, T056  
**Dependencies**: WP01, WP05 (collision), WP06 (case updates)  
**Parallel Opportunities**: T055, T056 can run in parallel after T054  
**Risks**: Connection management at scale; event ordering guarantees; reconnection logic; fallback to polling; memory leaks  
**Estimated Prompt Size**: ~350 lines  

---

### WP14 — Frontend Project Setup & Architecture

**Goal**: Initialize React + TypeScript + Vite with shared tooling  
**Priority**: Critical (blocks all frontend work)  
**Independent Test**: `npm run build && npm run test` passes; dev server starts; component structure in place  
**Subtasks**: T057, T058, T059  
**Dependencies**: None (can run parallel with WP01)  
**Parallel Opportunities**: T058, T059 can run in parallel after T057  
**Risks**: Shared Zod schemas with backend; Vite config for proxy; TypeScript strict mode  
**Estimated Prompt Size**: ~300 lines  

---

### WP15 — Customer Portal & Authentication UI

**Goal**: Customer-facing portal with auth and case management  
**Priority**: High  
**Independent Test**: Customer logs in, views own cases, creates case, communicates with support  
**Subtasks**: T060, T061  
**Dependencies**: WP02, WP04, WP14  
**Parallel Opportunities**: T061 can run after T060  
**Risks**: Auth state persistence; protected route guards; case creation UX; communication thread display  
**Estimated Prompt Size**: ~350 lines  

---

### WP16 — Agent Workspace

**Goal**: Agent-facing case editor with macros, shortcuts, collision UI  
**Priority**: High  
**Independent Test**: Agent views assigned/queue cases; edits case with collision warning; applies macros; keyboard shortcuts work; single-action reply+pending  
**Subtasks**: T062, T063  
**Dependencies**: WP04, WP05, WP06, WP14  
**Parallel Opportunities**: T063 can run in parallel after T062 core  
**Risks**: Real-time collision UI race conditions; macro picker UX; keyboard shortcut conflicts; optimistic locking UI  
**Estimated Prompt Size**: ~400 lines  

---

### WP17 — Supervisor & Admin UI

**Goal**: Supervisor dashboards and admin configuration interface  
**Priority**: High  
**Independent Test**: Supervisor sees workload/backlog/aging/SLA-risk; admin manages all config with validation; audit log viewable  
**Subtasks**: T064, T065, T066  
**Dependencies**: WP07, WP09, WP10, WP14  
**Parallel Opportunities**: T065, T066 can run in parallel after T064  
**Risks**: Complex dashboard data aggregation; config validation UX; audit log pagination/filtering; permission-aware UI  
**Estimated Prompt Size**: ~450 lines  

---

### WP18 — Real-Time Frontend Integration

**Goal**: Connect all frontend mutation operations to SSE for live updates  
**Priority**: High  
**Independent Test**: Case/queue changes reflect without refresh; collision warnings appear; graceful degradation works  
**Subtasks**: T067, T068  
**Dependencies**: WP13, WP15, WP16, WP17  
**Parallel Opportunities**: T068 can run in parallel after T067  
**Risks**: EventSource reconnection; memory leaks; event ordering; UI update flicker  
**Estimated Prompt Size**: ~300 lines  

---

### WP19 — CI/CD, Kubernetes & E2E Validation

**Goal**: Automated pipeline, K8s manifests, and end-to-end demo validation  
**Priority**: High (portfolio demonstration)  
**Independent Test**: GitHub Actions runs lint/typecheck/test/build; K8s manifests deploy; E2E tests pass; seed data works for demo  
**Subtasks**: T069, T070, T071, T072  
**Dependencies**: All prior WPs  
**Parallel Opportunities**: T070, T071, T072 can run in parallel after T069  
**Risks**: GitHub Actions secrets management; K8s manifest correctness; E2E flakiness; demo data realism  
**Estimated Prompt Size**: ~400 lines  

---

## Dependency Graph

```
WP01 (Foundation)
  ├─→ WP02 (Auth) → WP03 (Users/Customers)
  │                    │
  │                    └─→ WP04 (Case Core) ← WP12 (Async/BullMQ)
  │                                       │
  ├─→ WP14 (Frontend Setup)               ├─→ WP05 (Comms/Macros/Collision)
  │                                       │                    │
  │                                       ├─→ WP06 (Agent Ops) │
  │                                       │                    │
  │                                       ├─→ WP07 (Search)    │
  │                                       │                    │
  │                                       ├─→ WP08 (SLA) ←─────┘
  │                                       │
  │                                       ├─→ WP09 (Dashboards)
  │                                       │
  │                                       ├─→ WP10 (Admin Config)
  │                                       │
  │                                       ├─→ WP11 (Email) ←────┘
  │                                       │
  │                                       └─→ WP13 (SSE)
  │
  ├─→ WP15 (Customer Portal) ← WP02, WP04
  ├─→ WP16 (Agent Workspace) ← WP04, WP05, WP06
  ├─→ WP17 (Supervisor/Admin UI) ← WP07, WP09, WP10
  ├─→ WP18 (Real-time Frontend) ← WP13, WP15, WP16, WP17
  │
  └─→ WP19 (CI/CD/K8s/E2E) ← All WPs
```

---

## MVP Scope Recommendation

**Minimum Viable Demo (WP01-WP06, WP14-WP16)**: Core case lifecycle with auth, customer portal, agent workspace. Covers: case creation, routing, status transitions, communications, collision prevention, email-to-case.

**Full MVP (All 19 WPs)**: Adds SLA, dashboards, reporting, admin config, supervisor UI, real-time, CI/CD, K8s deployment.