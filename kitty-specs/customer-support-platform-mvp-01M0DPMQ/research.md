# Research Findings: Customer Support Platform MVP

**Mission**: customer-support-platform-mvp-01M0DPMQ  
**Date**: 2026-08-20  
**Phase**: 0 (Outline & Research)

---

## Technical Context Decisions (Resolved)

### Email Provider
- **Decision**: Postmark (free Developer plan for portfolio demo, isolated behind provider-independent boundary)
- **Rationale**: Postmark specializes in transactional email with excellent deliverability, simple webhook-based integration, and a generous free tier suitable for portfolio demonstrations. The provider-independent boundary (adapter pattern) allows switching providers without changing core logic.
- **Alternatives considered**: SendGrid (more marketing-focused), Mailgun (good but Postmark better for transactional), AWS SES (requires AWS account, more complex setup)

### CI/CD Platform
- **Decision**: GitHub Actions
- **Rationale**: Native to GitHub (where the repo lives), excellent TypeScript/Node.js support, built-in container building, easy Kubernetes deployment via actions, matrix testing support, free for public repos
- **Alternatives considered**: GitLab CI (would require GitLab), other (unnecessary complexity)

### Observability Stack
- **Decision**: Pino + Prometheus + OpenTelemetry
- **Rationale**: Pino is the fastest JSON logger for Node.js with low overhead. Prometheus is the industry standard for metrics with excellent Kubernetes integration. OpenTelemetry provides vendor-neutral tracing. All three are pure TypeScript/Node.js libraries with no external daemon requirements for development.
- **Alternatives considered**: Winston (slower, more complex), other combinations (would fragment the stack)

### Attachment Storage
- **Decision**: S3-compatible (MinIO for local development/demo, configurable endpoint for production S3-compatible provider)
- **Rationale**: MinIO provides full S3 API compatibility locally, enabling production-like development without cloud costs. Configurable endpoint allows seamless transition to AWS S3, Cloudflare R2, Backblaze B2, or any S3-compatible provider. Avoids database BLOB bloat and local filesystem container persistence issues.
- **Alternatives considered**: Local filesystem (not production-like, container persistence issues), Database BLOB (PostgreSQL bytea bloats DB, no CDN integration)

### Container Orchestration
- **Decision**: Both — Docker Compose for local development and Kubernetes manifests for live portfolio demonstration
- **Rationale**: Docker Compose offers zero-config local development with `docker-compose up`. Kubernetes manifests demonstrate production deployment skills for the portfolio. Base/overlay structure (kustomize) enables dev/prod parity with environment-specific values.
- **Alternatives considered**: Docker Compose only (misses K8s demo value), Kubernetes only (high local dev friction)

---

## Domain Research: Unresolved Rules & Invariants

### Case State Machine Invariants
**Research Task**: Clarify invariant, state transition, and event behavior for case lifecycle

**Findings**:
- **State machine**: New → Open → Pending Customer / Pending Internal → Resolved → Closed (from spec)
- **SLA timer behavior**: Pauses in Pending Customer, resumes on return to Open (FR-072)
- **Resolved transition requirements**: Requires validated Root Cause, Category, and Final Priority (FR-034)
- **Inbound email auto-transition**: Pending Customer → Open on customer reply (FR-032)
- **Catch-all queue**: Mandatory system queue for routing failures, immutable (FR-023, FR-095)
- **Administrative alert**: Generated when routing fails or deprecated config encountered (FR-023)

**Key Invariants**:
1. A case cannot transition to Resolved without Root Cause, Category, and Final Priority populated and validated
2. SLA resolution timer MUST pause in Pending Customer and resume in Open — no exceptions
3. Catch-all queue can never be deactivated or deleted (FR-095)
4. Configuration referenced by active (non-Closed) cases cannot be deleted (FR-093)
5. Queue/status/category deactivation blocked until active cases reassigned (FR-094)

**Event Behavior**:
- Assignment changes → always audit (FR-102)
- Status changes → always audit (FR-102)
- Escalation events → recorded in case history (FR-075)
- Real-time updates for queue/case changes (FR-110, FR-111)

### Email-to-Case Ingestion Rules
**Research Task**: Clarify email parsing, association, and provider boundary behavior

**Findings**:
- **Ingestion**: Webhook-based, provider-independent boundary (C-014, FR-025)
- **Association**: Inbound email response associated with existing case when valid relationship determined (FR-026)
- **Provider**: Postmark selected, free Developer plan
- **Adapter pattern**: PostmarkAdapter implements EmailProvider interface for send/receive/webhook verification

**Key Invariants**:
1. Email ingestion MUST not create duplicate cases for same thread
2. Webhook signature verification REQUIRED for security
3. Failed ingestion MUST not lose emails — dead letter queue with admin alert
4. Outbound notifications use same provider-independent boundary

### Real-Time Collision Prevention
**Research Task**: Clarify real-time drafting lock behavior and edge cases

**Findings**:
- **Trigger**: When agent actively drafting response or internal note (FR-039)
- **Warning**: Other users receive real-time warning
- **Lock**: Response editor temporarily locked against simultaneous editing
- **Transport**: SSE for server→client push (per Technical Context)

**Key Invariants**:
1. Lock MUST be released on: draft submission, navigation away, timeout, or connection loss
2. Warning MUST be delivered via SSE in <100ms p95 (per Performance Goals)
3. Graceful degradation: if SSE unavailable, fall back to optimistic locking with conflict detection on save (NFR-011)

### SLA Pause/Resume Edge Cases
**Research Task**: Clarify SLA timer behavior during edge transitions

**Findings**:
- **Pause**: Only in Pending Customer status (FR-072)
- **Resume**: On transition back to Open (FR-072)
- **Other statuses**: Pending Internal does NOT pause SLA (only Pending Customer)
- **Breach detection**: Background job monitors approaching/exceeding targets (FR-073)

**Key Invariants**:
1. Timer state (running/paused) MUST be persisted with case
2. Timer calculation MUST account for all pause/resume cycles
3. Configuration changes to SLA rules MUST NOT retroactively alter in-flight timers
4. Escalation event recorded at breach (FR-075)

### Configuration Referential Integrity
**Research Task**: Clarify deletion/deactivation constraints for admin config

**Findings**:
- **FR-093**: Prevent deletion of config records referenced by historical cases
- **FR-094**: Prevent deactivation of queue/status/category assigned to active (non-Closed) cases until reassigned
- **FR-095**: Catch-all queue immutable — cannot deactivate or delete

**Implementation Approach**:
- Soft delete with `deleted_at` timestamp for most config
- Hard delete ONLY if zero references exist (check via FK or application query)
- Deactivation = `is_active = false` with validation query against active cases
- Catch-all queue: `is_system = true` with DB constraint preventing deletion/deactivation

---

## Technology Best Practices Research

### Bull/BullMQ for Async Processing
- **Pattern**: Queue per job type (sla-monitor, email-delivery, report-generation, audit-write)
- **Retry**: Exponential backoff with max attempts, dead letter queue after exhaustion
- **Priority**: Use priority queues for SLA breach alerts (high) vs report generation (low)
- **Metrics**: BullMQ built-in metrics + Prometheus exporter
- **Scaling**: Horizontal workers via Kubernetes HPA on queue depth

### PostgreSQL Full-Text Search
- **Implementation**: `tsvector` column on cases table, GIN index, `tsquery` for search
- **Weighting**: Subject (A), Description (B), Customer name (C), Case number (A)
- **Ranking**: `ts_rank_cd` with normalization
- **Language**: `english` config, consider `simple` for case numbers
- **Triggers**: Auto-update tsvector on case/communication insert/update

### React + TypeScript Frontend Architecture
- **State**: React Context for auth/realtime, React Query (TanStack Query) for server state
- **Components**: Atomic design — atoms/molecules/organisms/pages
- **Forms**: React Hook Form + Zod (shared schemas with backend)
- **Real-time**: Custom `useRealtime` hook wrapping EventSource/SSE
- **Routing**: React Router v6 with lazy-loaded page chunks

### Express/Fastify Backend Architecture
- **Framework**: Fastify preferred (better performance, TypeScript-native, schema validation)
- **Validation**: Zod schemas shared with frontend via `packages/shared` or copied
- **Database**: Prisma ORM (type-safe, migrations, PostgreSQL full-text support)
- **Auth**: JWT middleware with refresh token rotation, RBAC middleware per route
- **Error handling**: Standardized error codes, structured logging with correlation IDs

---

## Integration Patterns

### Email Provider Adapter Interface
```typescript
interface EmailProvider {
  send(params: SendEmailParams): Promise<SendResult>;
  verifyWebhookSignature(payload: string, signature: string): boolean;
  parseInboundWebhook(payload: unknown): InboundEmail;
}
```

### SSE Event Format
```typescript
interface SSEEvent<T> {
  event: string;        // e.g., "case.updated", "queue.changed", "collision.warning"
  data: T;              // Typed payload
  timestamp: string;    // ISO 8601
  correlationId: string;
}
```

### S3 Storage Abstraction
```typescript
interface StorageProvider {
  putObject(key: string, body: Buffer, contentType: string): Promise<PutResult>;
  getSignedUrl(key: string, expiresIn: number): Promise<string>;
  deleteObject(key: string): Promise<void>;
}
```

---

## Risk Register (from Implementation Concerns)

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Real-time collision race conditions | High | High | Deterministic lock acquisition, SSE delivery guarantees, fallback optimistic locking |
| SLA timer accuracy across restarts | Medium | High | Persist timer state, recalculate on startup, idempotent monitor job |
| PostgreSQL full-text search performance | Medium | Medium | GIN indexes, query optimization, consider pg_trgm for trigram fallback |
| Attachment storage cost/scale | Low | Medium | MinIO locally, lifecycle policies in prod, CDN for downloads |
| Audit log storage growth | Medium | Medium | Partitioning, retention policy, compression |
| Configuration change propagation | Medium | High | Event-driven cache invalidation, versioned config reads |
| Webhook security (email ingestion) | High | High | Signature verification, rate limiting, IP allowlist option |
| JWT refresh token rotation edge cases | Medium | High | Short access tokens (15min), refresh rotation, revocation list |

---

## Summary

All Technical Context NEEDS CLARIFICATION items have been resolved with concrete decisions. Domain invariants and edge cases have been researched and documented. Technology choices align with constraints (NFR-001 through NFR-011, C-001 through C-016). Ready for Phase 1 design artifacts.