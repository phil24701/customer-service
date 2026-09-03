# Customer Support Platform MVP — Specification

## 1. Executive Summary

**Purpose**: Build a production-style customer support case management platform with full case lifecycle, email ingestion, routing, SLA management, real-time collaboration, and role-based administration.

**Scope**: Complete MVP covering 17 validated items including authentication, case lifecycle, email-to-case, routing/queues, SLA, real-time updates, macros, bulk actions, audit logging, and admin configuration. Implementation phased starting with Epic 1 (Authentication & Access).

## 2. User Roles

| Role | Description |
|------|-------------|
| **Customer** | Submit cases, view own cases, communicate with support, receive status updates |
| **Support Agent** | Work cases, communicate with customers, add internal notes, apply macros, manage case fields, record resolution |
| **Support Supervisor** | Monitor queues/workload, identify SLA risk, reassign cases, manage escalations, review metrics |
| **Administrator** | Manage users, roles, queues, categories, priorities, statuses, service-level rules, system configuration |

## 3. Functional Requirements

### 3.1 Authentication & Authorization

| ID | Requirement | Status |
|----|-------------|--------|
| FR-001 | The system shall authenticate registered users using email address and password | Accepted |
| FR-002 | The system shall support role-based authorization for Customer, Support Agent, Support Supervisor, and Administrator roles | Accepted |
| FR-003 | The system shall prevent users from accessing functions or records outside their authorization scope | Accepted |
| FR-004 | Administrators shall be able to create, edit, deactivate, and reactivate user accounts | Accepted |
| FR-005 | Administrators shall be able to assign applicable roles to users | Accepted |
| FR-006 | The system shall record significant administrative changes with actor and timestamp | Accepted |

### 3.2 Customer Management

| ID | Requirement | Status |
|----|-------------|--------|
| FR-010 | Authorized users shall be able to search customers by name, email, organization, and customer identifier | Accepted |
| FR-011 | Support users shall be able to view a customer profile and case history | Accepted |
| FR-012 | Authorized users shall be able to create and update customer records | Accepted |
| FR-013 | A customer shall be able to view only cases associated with that customer | Accepted |
| FR-014 | The system shall support an optional organization relationship for customers | Accepted |

### 3.3 Case Lifecycle

| ID | Requirement | Status |
|----|-------------|--------|
| FR-020 | A customer shall be able to submit a case with subject, description, category, and optional attachment | Accepted |
| FR-021 | Authorized support users shall be able to create a case on behalf of a customer | Accepted |
| FR-022 | The system shall generate a unique human-readable case number | Accepted |
| FR-023 | The system shall generate default status, priority, and queue values according to configured rules. If routing or rule evaluation fails, or a deprecated configuration is encountered, the case shall be routed to a mandatory system catch-all queue and an administrative alert shall be generated | Accepted |
| FR-024 | The system shall record case creation timestamp and creator | Accepted |
| FR-025 | The system shall support email-to-case ingestion for incoming customer email | Accepted |
| FR-026 | The system shall associate an inbound email response with the appropriate existing case when a valid case relationship can be determined | Accepted |

**Case Statuses**: New → Open → Pending Customer / Pending Internal → Resolved → Closed
- SLA resolution timers pause in Pending Customer and resume on return to Open
- Transition to Resolved requires validated Root Cause, Category, and Final Priority

### 3.4 Agent Case Management

| ID | Requirement | Status |
|----|-------------|--------|
| FR-030 | Agents shall be able to view cases assigned to themselves and cases in queues to which they have access | Accepted |
| FR-031 | Authorized users shall be able to assign or reassign a case to an agent and/or queue | Accepted |
| FR-032 | Agents shall be able to change case status. An inbound customer communication shall automatically transition a case from Pending Customer to Open | Accepted |
| FR-033 | Authorized agents shall be able to change case priority and category | Accepted |
| FR-034 | The system shall support New, Open, Pending Customer, Pending Internal, Resolved, and Closed statuses. Transitioning a case to Resolved shall require populated and validated Root Cause, Category, and Final Priority information | Accepted |
| FR-035 | The system shall maintain a chronological case activity history | Accepted |
| FR-036 | The system shall record previous and new values for important case-field changes | Accepted |
| FR-037 | Supervisors shall be able to reassign cases and override normal assignment restrictions | Accepted |
| FR-038 | The system shall support case escalation as a distinct operational event | Accepted |
| FR-039 | The system shall provide real-time agent collision prevention. When an agent is actively drafting a response or internal note, other users shall receive a real-time warning and the response editor shall be temporarily locked against simultaneous editing | Accepted |
| FR-040 | Agents shall be able to send a customer response and set the case to Pending Customer as a single action | Accepted |
| FR-041 | When a case is Pending Internal, the system shall require identification of the responsible internal department or owner | Accepted |

### 3.5 Communications

| ID | Requirement | Status |
|----|-------------|--------|
| FR-045 | Customers and agents shall be able to exchange case messages | Accepted |
| FR-046 | Agents shall be able to add internal notes that are not visible to customers | Accepted |
| FR-047 | The system shall distinguish customer-visible messages from internal notes | Accepted |
| FR-048 | The case view shall display communications in chronological order | Accepted |
| FR-049 | Users shall be able to attach supported files to case communications subject to configured limits | Accepted |
| FR-050 | The system shall record message author and timestamp | Accepted |
| FR-051 | Authorized users shall be able to apply preconfigured message templates or macros to communications | Accepted |
| FR-052 | Macros may populate message text and update applicable case fields | Accepted |
| FR-053 | The system shall support keyboard shortcuts for selected core triage actions, including submitting a response, toggling between reply and internal note, and applying a macro | Accepted |

### 3.6 Search, Filtering & Queue Views

| ID | Requirement | Status |
|----|-------------|--------|
| FR-060 | Support users shall be able to search cases by case number, subject, customer, and text content | Accepted |
| FR-061 | Support users shall be able to filter cases by status, priority, category, queue, assigned agent, internal department/owner, organization, and date range | Accepted |
| FR-062 | Queue views shall display cases awaiting work in that queue | Accepted |
| FR-063 | Case lists shall support sorting and pagination | Accepted |
| FR-064 | The system shall provide supervisor views of unassigned and overdue cases | Accepted |
| FR-065 | Authorized users shall be able to perform basic bulk actions from queue views, including bulk assignment and bulk status changes | Accepted |
| FR-066 | Bulk actions shall enforce the same authorization and business rules that apply to equivalent individual case actions | Accepted |

### 3.7 SLA Management

| ID | Requirement | Status |
|----|-------------|--------|
| FR-070 | Administrators shall be able to configure response and/or resolution targets by applicable category, priority, or queue | Accepted |
| FR-071 | The system shall calculate target response and resolution timestamps for applicable cases | Accepted |
| FR-072 | SLA resolution timers shall automatically pause while a case is in Pending Customer and resume when the case returns to Open | Accepted |
| FR-073 | The system shall identify cases approaching or exceeding service targets | Accepted |
| FR-074 | Supervisors shall be able to view cases at risk of or in breach of a service target | Accepted |
| FR-075 | The system shall record escalation events in case history | Accepted |

### 3.8 Dashboards & Reporting

| ID | Requirement | Status |
|----|-------------|--------|
| FR-080 | Agents shall have a dashboard showing assigned/open cases and cases requiring attention | Accepted |
| FR-081 | Supervisors shall have a dashboard showing case volume, workload by agent, queue backlog, aging, and overdue cases | Accepted |
| FR-082 | The system shall provide case-volume reporting by status, priority, category, and time period | Accepted |
| FR-083 | The system shall provide average response and resolution metrics where sufficient data exists | Accepted |
| FR-084 | Authorized users shall be able to drill from summary metrics to underlying case lists | Accepted |
| FR-085 | Reports shall respect viewer authorization scope | Accepted |

### 3.9 Administration & Configuration

| ID | Requirement | Status |
|----|-------------|--------|
| FR-090 | Administrators shall be able to manage case statuses, priorities, categories, and queues | Accepted |
| FR-091 | Administrators shall be able to configure applicable administrative and case-management permissions | Accepted |
| FR-092 | Administrators shall be able to manage service-level rules | Accepted |
| FR-093 | The system shall prevent deletion of configuration records referenced by historical cases | Accepted |
| FR-094 | The system shall prevent deactivation of any queue, status, or category currently assigned to active (non-Closed) cases until those cases have been appropriately reassigned or otherwise made valid | Accepted |
| FR-095 | The system shall maintain an immutable system catch-all queue that cannot be deactivated or deleted | Accepted |
| FR-096 | Administrators shall be able to review relevant audit events | Accepted |

### 3.10 Audit Logging

| ID | Requirement | Status |
|----|-------------|--------|
| FR-100 | The system shall maintain an audit trail for significant case changes, assignments, status changes, escalations, and administrative configuration changes | Accepted |
| FR-101 | Audit records shall include event type, actor, timestamp, and relevant before/after information where applicable | Accepted |
| FR-102 | Assignment and status changes shall always generate audit events | Accepted |
| FR-103 | Normal users shall not be able to alter or delete audit history | Accepted |

### 3.11 Real-Time Updates

| ID | Requirement | Status |
|----|-------------|--------|
| FR-110 | Relevant case and queue changes shall be reflected to authorized users without requiring a manual page refresh | Accepted |
| FR-111 | Real-time updates shall include changes relevant to queue triage, assignment, status, and other operational events as defined by the technical design | Accepted |

## 4. Non-Functional Requirements

| ID | Requirement | Status |
|----|-------------|--------|
| NFR-001 | The application shall use a layered architecture separating presentation, application/business logic, and persistence concerns | Accepted |
| NFR-002 | APIs shall validate input and enforce authorization server-side | Accepted |
| NFR-003 | The database shall enforce referential integrity and use appropriate indexes | Accepted |
| NFR-004 | The application shall provide clear validation and error handling | Accepted |
| NFR-005 | The system shall provide application logging and operational diagnostics | Accepted |
| NFR-006 | The application shall be deployable using reproducible configuration suitable for a portfolio demonstration | Accepted |
| NFR-007 | Secrets and credentials shall be externalized from source code | Accepted |
| NFR-008 | The project shall include realistic seed/demo data for major workflows | Accepted |
| NFR-009 | Principal business operations shall be accessible through documented RESTful APIs | Accepted |
| NFR-010 | The UI shall support common desktop browser sizes and adapt reasonably to smaller screens | Accepted |
| NFR-011 | Real-time mechanisms shall fail gracefully without compromising case integrity if the real-time channel is unavailable | Accepted |

## 5. Constraints

| ID | Constraint | Status |
|----|------------|--------|
| C-001 | Frontend: React + TypeScript (portfolio requirement) | Accepted |
| C-002 | Backend: Node.js + TypeScript (preferred direction) | Accepted |
| C-003 | API: REST (primary application API style) | Accepted |
| C-004 | Database: PostgreSQL (system of record) | Accepted |
| C-005 | Caching/transient state: Redis (where technically justified) | Accepted |
| C-006 | Search: PostgreSQL full-text search and/or dedicated search (to be determined in technical design) | Accepted |
| C-007 | Asynchronous processing: Background jobs (mechanism to be selected during technical design) | Accepted |
| C-008 | Real-time: WebSockets or Server-Sent Events (to be selected during technical design based on communication patterns) | Accepted |
| C-009 | Authentication/Authorization: Application authentication with RBAC using JWT tokens with refresh tokens | Accepted |
| C-010 | Testing: Unit + integration + end-to-end (frameworks to be selected) | Accepted |
| C-011 | Containers: Docker (primary packaging/deployment approach) | Accepted |
| C-012 | CI/CD: Automated repository-based CI/CD (platform details to be selected) | Accepted |
| C-013 | Observability: Structured logging plus practical health/diagnostic monitoring (tooling to be selected) | Accepted |
| C-014 | External integration: Email integration via webhook-based provider-independent boundary (provider selected during technical design) | Accepted |
| C-015 | AI: Bounded AI-assisted support features (classification, summaries, suggested responses) — excluded from MVP per scope review | Accepted |
| C-016 | Multi-tenant: Credentials authenticated before tenant disclosure; multi-tenant users select tenant after authentication | Accepted |

## 6. Key Decisions (Resolved During Discovery)

| Decision | Resolution |
|----------|------------|
| **Mission Scope** | Full MVP implemented in single mission; phased implementation starting with Epic 1 (Authentication) |
| **Auth Mechanism** | JWT tokens with refresh tokens (stateless, scalable, access + refresh token pattern) |
| **Real-Time Mechanism** | Hybrid — SSE for server→client push (case/queue updates, notifications) + REST for client→server mutations (DM-01M0DXQSVZWTHJK68HTV5R2JX9) |
| **Async Mechanism** | Bull/BullMQ (Redis-based) — mature queue with retries, scheduling, priority, metrics, horizontal scaling (DM-01M0DWBJ0X7MCF06FFX46S6REE) |
| **Email Integration** | Webhook-based with provider-independent boundary; Postmark selected (free Developer plan for portfolio demo) (DM-01M0E1QJM5DRB1CV9NQCBN78B9) |
| **Backend Framework** | NestJS (opinionated, modular, DI, good for enterprise) (DM-01M0DVBY0GRFRQ5M7QCJC7TJQJ) |
| **ORM** | Prisma (type-safe, migrations, PostgreSQL full-text support) (DM-01M0DVGNRRX1SYKGNA56FHK8WB) |
| **Search Implementation** | PostgreSQL full-text search (built-in, no extra infrastructure) — leverages existing PostgreSQL with tsvector/tsquery, GIN indexes (DM-01M0DZMVM8R813073P7TDAKTZQ) |
| **CI/CD Platform** | GitHub Actions (native to GitHub, excellent TypeScript/Node.js support) (DM-01M0GHZGT8VSP2DAWEF0CVG5R1) |
| **Observability Stack** | Pino + Prometheus + OpenTelemetry (DM-01M0GJ3PGAKRYQFERND4D7W3XZ) |
| **Attachment Storage** | S3-compatible (MinIO for local development/demo, configurable endpoint for production S3-compatible provider) (DM-01M0GKP5W8ZYKH6GBV3J451F71) |
| **Container Orchestration** | Both — Docker Compose for local development and Kubernetes manifests for live portfolio demonstration (DM-01M0GKZ76P86A3XBNQW39S4FKX) |

## 7. Deferred / Future Scope (Explicitly Not MVP)

- Pinned Internal Case Summary
- Dedicated Cross-Department Handoff Tracking and reporting
- Customer Satisfaction / CSAT surveys
- Knowledge Base
- AI-Assisted Support (classification, summaries, suggested responses)

## 8. Success Criteria

- A customer can create and follow a case through the portal or email
- An agent can efficiently triage, communicate, assign, update, and resolve cases
- Agents are protected against simultaneous response editing
- Customer replies automatically reopen cases requiring support attention
- Routing failures cannot silently lose cases (catch-all queue + admin alert)
- Internal handoffs identify the department or owner responsible for the next action
- SLA measurements pause while waiting for customer input
- Supervisors can identify workload, SLA risk, overdue cases, and unassigned work
- Agents can perform high-volume operations using macros, keyboard shortcuts, and bulk actions
- Assignment and status changes are auditable
- Real-time case and queue changes are visible without manual refresh
- The platform can ingest cases through email and support realistic end-to-end demonstrations

## 9. Assumptions

- Single-tenant demo deployment is sufficient for portfolio demonstration; multi-tenant architecture is built but not exercised in demo
- Administrator-created/demo accounts are sufficient; self-service customer registration is out of scope
- Exact permission matrix for roles is an implementation/design decision
- Attachment storage strategy (local, S3-compatible, database) to be determined in technical design
- Specific search implementation (PostgreSQL full-text vs dedicated) to be determined in technical design
- Container orchestration for demo deployment (Docker Compose vs Kubernetes) to be determined

## 10. Domain Language

| Canonical Term | Definition | Avoid / Synonyms |
|----------------|------------|------------------|
| **Case** | Primary work item representing a customer issue or request | Ticket, Issue, Request |
| **Queue** | Logical grouping of cases awaiting work by a set of agents | Bucket, Pool, List |
| **Macro** | Preconfigured message template that may also update case fields | Canned Response, Template, Snippet |
| **SLA** | Service Level Agreement — configured response/resolution targets | Service Target, KPI |
| **Catch-All Queue** | Mandatory system queue receiving cases when routing fails | Fallback Queue, Unassigned Queue |
| **Pending Customer** | Case status awaiting customer response; SLA timer paused | Waiting on Customer |
| **Pending Internal** | Case status awaiting internal department/owner action | Waiting on Internal, Escalated |
| **Internal Department / Owner** | Identified internal team responsible for a case in Pending Internal | Internal Owner, Handoff Target |
| **Root Cause** | Required resolution field categorizing why the issue occurred | Resolution Code, Close Reason |