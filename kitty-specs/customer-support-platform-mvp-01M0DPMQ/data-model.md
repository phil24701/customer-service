# Data Model: Customer Support Platform MVP

**Mission**: customer-support-platform-mvp-01M0DPMQ  
**Date**: 2026-08-20  
**Phase**: 1 (Design & Contracts)

---

## Entity Relationship Overview

```
┌─────────────┐       ┌─────────────┐       ┌─────────────┐
│   Tenant    │───────│    User     │───────│  Customer   │
└─────────────┘       └─────────────┘       └─────────────┘
                            │                     │
                            │                     │
                     ┌──────┴──────┐       ┌──────┴──────┐
                     ▼             ▼       ▼             ▼
               ┌───────────┐ ┌───────────┐ ┌─────────┐ ┌──────────┐
               │ UserRole  │ │  Session  │ │  Case   │ │Organization│
               └───────────┘ └───────────┘ └─────────┘ └──────────┘
                     │             │       │    │
                     │             │       │    │
              ┌──────┴──────┐ ┌─────┴─────┐ │    │
              ▼             ▼ ▼           ▼ ▼    ▼
         ┌──────────┐ ┌──────────┐ ┌────────┐ ┌────────┐
         │  Queue   │ │ Category │ │ Status │ │Priority│
         └──────────┘ └──────────┘ └────────┘ └────────┘
              │             │
              │             │
        ┌─────┴─────┐ ┌─────┴─────┐
        ▼           ▼ ▼           ▼
   ┌────────┐ ┌────────┐    ┌──────────┐
   │  Case  │ │  SLA   │    │  Macro   │
   └────────┘ └────────┘    └──────────┘
        │
        │
┌───────┼───────┐
▼       ▼       ▼
┌─────┐ ┌─────┐ ┌─────────┐
│Comm │ │Audit│ │Attachmnt│
└─────┘ └─────┘ └─────────┘
```

---

## Core Entities

### Tenant
**Purpose**: Multi-tenant isolation root (single-tenant demo, but architected for multi-tenant)

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | Unique identifier |
| name | VARCHAR(255) | NOT NULL, UNIQUE | Tenant display name |
| slug | VARCHAR(100) | NOT NULL, UNIQUE | URL-safe identifier |
| is_active | BOOLEAN | NOT NULL, DEFAULT true | Soft delete flag |
| settings | JSONB | DEFAULT '{}' | Tenant-level configuration |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() | |
| updated_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() | |

**Indexes**: `UNIQUE (slug)`, `INDEX (is_active)`

---

### User
**Purpose**: System users (customers, agents, supervisors, admins)

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | |
| tenant_id | UUID | NOT NULL, FK → Tenant.id | Tenant isolation |
| email | VARCHAR(255) | NOT NULL, UNIQUE per tenant | Login identifier |
| password_hash | VARCHAR(255) | NOT NULL | Argon2id hash |
| full_name | VARCHAR(255) | NOT NULL | Display name |
| avatar_url | VARCHAR(500) | NULLABLE | Profile image |
| timezone | VARCHAR(50) | NOT NULL, DEFAULT 'UTC' | IANA timezone |
| locale | VARCHAR(10) | NOT NULL, DEFAULT 'en-US' | Language preference |
| is_active | BOOLEAN | NOT NULL, DEFAULT true | Soft delete / deactivation |
| last_login_at | TIMESTAMPTZ | NULLABLE | |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() | |
| updated_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() | |

**Indexes**: `UNIQUE (tenant_id, email)`, `INDEX (tenant_id, is_active)`, `INDEX (created_at)`

---

### UserRole
**Purpose**: Role assignments per user (many-to-many User ↔ Role)

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | |
| user_id | UUID | NOT NULL, FK → User.id | |
| role | VARCHAR(50) | NOT NULL | Enum: 'customer', 'agent', 'supervisor', 'admin' |
| tenant_id | UUID | NOT NULL, FK → Tenant.id | For cross-tenant admin (future) |
| assigned_by | UUID | FK → User.id | Audit trail |
| assigned_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() | |

**Indexes**: `UNIQUE (user_id, role)`, `INDEX (tenant_id, role)`

**Roles** (canonical per spec):
- `customer` — Submit/view own cases, communicate, receive updates
- `agent` — Work assigned cases, communicate, macros, bulk actions
- `supervisor` — Monitor queues, reassign, escalate, metrics
- `admin` — Full configuration, user management, audit review

---

### Session
**Purpose**: JWT refresh token storage for invalidation

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | |
| user_id | UUID | NOT NULL, FK → User.id | |
| refresh_token_hash | VARCHAR(255) | NOT NULL, UNIQUE | SHA-256 of refresh token |
| user_agent | TEXT | NULLABLE | Client identification |
| ip_address | INET | NULLABLE | |
| expires_at | TIMESTAMPTZ | NOT NULL | |
| revoked_at | TIMESTAMPTZ | NULLABLE | Rotation/revocation |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() | |

**Indexes**: `UNIQUE (refresh_token_hash)`, `INDEX (user_id, revoked_at)`, `INDEX (expires_at)`

---

### Organization
**Purpose**: Optional customer grouping (FR-014)

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | |
| tenant_id | UUID | NOT NULL, FK → Tenant.id | |
| name | VARCHAR(255) | NOT NULL | |
| domain | VARCHAR(255) | NULLABLE, UNIQUE per tenant | Email domain for auto-association |
| is_active | BOOLEAN | NOT NULL, DEFAULT true | |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() | |
| updated_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() | |

**Indexes**: `UNIQUE (tenant_id, domain) WHERE domain IS NOT NULL`, `INDEX (tenant_id, is_active)`

---

### Customer
**Purpose**: Case submitters (may be Users with customer role, or external)

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | |
| tenant_id | UUID | NOT NULL, FK → Tenant.id | |
| user_id | UUID | NULLABLE, FK → User.id, UNIQUE | Link to User if registered |
| email | VARCHAR(255) | NOT NULL | Primary contact |
| full_name | VARCHAR(255) | NOT NULL | |
| phone | VARCHAR(50) | NULLABLE | |
| organization_id | UUID | NULLABLE, FK → Organization.id | Optional (FR-014) |
| external_id | VARCHAR(100) | NULLABLE, UNIQUE per tenant | CRM/external reference |
| is_active | BOOLEAN | NOT NULL, DEFAULT true | |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() | |
| updated_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() | |

**Indexes**: `UNIQUE (tenant_id, email)`, `INDEX (tenant_id, organization_id)`, `INDEX (tenant_id, external_id) WHERE external_id IS NOT NULL`

---

### Queue
**Purpose**: Logical grouping of cases awaiting work (FR-090, FR-095)

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | |
| tenant_id | UUID | NOT NULL, FK → Tenant.id | |
| name | VARCHAR(100) | NOT NULL | Display name |
| slug | VARCHAR(50) | NOT NULL, UNIQUE per tenant | URL/API identifier |
| description | TEXT | NULLABLE | |
| is_catch_all | BOOLEAN | NOT NULL, DEFAULT false | Immutable system queue (FR-095) |
| is_active | BOOLEAN | NOT NULL, DEFAULT true | Deactivation blocked if active cases (FR-094) |
| sla_rule_id | UUID | NULLABLE, FK → SLARule.id | Default SLA for queue |
| auto_assign | BOOLEAN | NOT NULL, DEFAULT false | Round-robin auto-assignment |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() | |
| updated_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() | |

**Indexes**: `UNIQUE (tenant_id, slug)`, `INDEX (tenant_id, is_active)`, `INDEX (is_catch_all) WHERE is_catch_all`

**Constraint**: Only ONE catch-all queue per tenant (enforced by application logic + partial unique index)

---

### Category
**Purpose**: Case categorization (FR-090)

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | |
| tenant_id | UUID | NOT NULL, FK → Tenant.id | |
| name | VARCHAR(100) | NOT NULL | |
| slug | VARCHAR(50) | NOT NULL, UNIQUE per tenant | |
| description | TEXT | NULLABLE | |
| color | VARCHAR(7) | NOT NULL, DEFAULT '#6366f1' | Hex color for UI |
| is_active | BOOLEAN | NOT NULL, DEFAULT true | Deactivation blocked if active cases (FR-094) |
| parent_id | UUID | NULLABLE, FK → Category.id | Hierarchical categories |
| sort_order | INTEGER | NOT NULL, DEFAULT 0 | Display ordering |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() | |
| updated_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() | |

**Indexes**: `UNIQUE (tenant_id, slug)`, `INDEX (tenant_id, is_active, parent_id)`

---

### Status
**Purpose**: Case lifecycle states (FR-034, FR-090)

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | |
| tenant_id | UUID | NOT NULL, FK → Tenant.id | |
| name | VARCHAR(50) | NOT NULL | Canonical: 'new', 'open', 'pending_customer', 'pending_internal', 'resolved', 'closed' |
| label | VARCHAR(100) | NOT NULL | Display label |
| description | TEXT | NULLABLE | |
| color | VARCHAR(7) | NOT NULL | Hex color |
| is_system | BOOLEAN | NOT NULL, DEFAULT false | Core statuses cannot be deleted |
| is_closed | BOOLEAN | NOT NULL, DEFAULT false | Terminal state (resolved/closed) |
| pauses_sla | BOOLEAN | NOT NULL, DEFAULT false | Only 'pending_customer' = true (FR-072) |
| requires_root_cause | BOOLEAN | NOT NULL, DEFAULT false | Only 'resolved' = true (FR-034) |
| sort_order | INTEGER | NOT NULL, DEFAULT 0 | |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() | |
| updated_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() | |

**Indexes**: `UNIQUE (tenant_id, name)`, `INDEX (tenant_id, is_system)`

**System Statuses** (seeded, is_system=true):
1. `new` — Initial state, pauses_sla=false
2. `open` — Active work, pauses_sla=false
3. `pending_customer` — Awaiting customer, pauses_sla=true
4. `pending_internal` — Awaiting internal, pauses_sla=false
5. `resolved` — Resolved, requires_root_cause=true, is_closed=false
6. `closed` — Terminal, is_closed=true

---

### Priority
**Purpose**: Case urgency levels (FR-090)

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | |
| tenant_id | UUID | NOT NULL, FK → Tenant.id | |
| name | VARCHAR(50) | NOT NULL | Canonical: 'low', 'medium', 'high', 'critical' |
| label | VARCHAR(100) | NOT NULL | |
| color | VARCHAR(7) | NOT NULL | |
| sort_order | INTEGER | NOT NULL, DEFAULT 0 | 1=critical, 2=high, 3=medium, 4=low |
| sla_rule_id | UUID | NULLABLE, FK → SLARule.id | Default SLA for priority |
| is_active | BOOLEAN | NOT NULL, DEFAULT true | |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() | |
| updated_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() | |

**Indexes**: `UNIQUE (tenant_id, name)`, `INDEX (tenant_id, is_active)`

---

### Case
**Purpose**: Primary work item (FR-020 through FR-041)

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | |
| tenant_id | UUID | NOT NULL, FK → Tenant.id | |
| case_number | VARCHAR(50) | NOT NULL, UNIQUE per tenant | Human-readable: CSP-2026-000123 |
| subject | VARCHAR(500) | NOT NULL | |
| description | TEXT | NOT NULL | Initial description |
| customer_id | UUID | NOT NULL, FK → Customer.id | |
| organization_id | UUID | NULLABLE, FK → Organization.id | Denormalized from customer |
| status_id | UUID | NOT NULL, FK → Status.id | Current status |
| priority_id | UUID | NOT NULL, FK → Priority.id | Current priority |
| category_id | UUID | NULLABLE, FK → Category.id | |
| queue_id | UUID | NOT NULL, FK → Queue.id | Current queue |
| assigned_agent_id | UUID | NULLABLE, FK → User.id | Assigned agent |
| internal_owner_id | UUID | NULLABLE, FK → User.id | For pending_internal (FR-041) |
| internal_department | VARCHAR(100) | NULLABLE | For pending_internal (FR-041) |
| root_cause | VARCHAR(100) | NULLABLE | Required for resolved (FR-034) |
| final_priority_id | UUID | NULLABLE, FK → Priority.id | Required for resolved (FR-034) |
| source | VARCHAR(20) | NOT NULL, DEFAULT 'portal' | 'portal', 'email', 'api' |
| email_message_id | VARCHAR(255) | NULLABLE | For email-to-case threading |
| email_thread_id | VARCHAR(255) | NULLABLE | For email threading |
| sla_target_response_at | TIMESTAMPTZ | NULLABLE | Calculated response target |
| sla_target_resolution_at | TIMESTAMPTZ | NULLABLE | Calculated resolution target |
| sla_paused_at | TIMESTAMPTZ | NULLABLE | When SLA timer paused |
| sla_total_paused_duration | INTERVAL | NOT NULL, DEFAULT '0' | Accumulated pause time |
| created_by_id | UUID | NOT NULL, FK → User.id | Creator (customer or agent) |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() | |
| updated_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() | |
| closed_at | TIMESTAMPTZ | NULLABLE | When status=closed |
| resolved_at | TIMESTAMPTZ | NULLABLE | When status=resolved |

**Indexes**:
- `UNIQUE (tenant_id, case_number)`
- `INDEX (tenant_id, status_id, queue_id)`
- `INDEX (tenant_id, assigned_agent_id, status_id)`
- `INDEX (tenant_id, customer_id, created_at DESC)`
- `INDEX (tenant_id, sla_target_resolution_at) WHERE sla_target_resolution_at IS NOT NULL`
- `INDEX (tenant_id, email_thread_id) WHERE email_thread_id IS NOT NULL`
- **Full-text search**: `tsvector` column `search_vector` with GIN index, trigger on subject/description/customer name

---

### Communication
**Purpose**: Messages and internal notes on cases (FR-045 through FR-050)

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | |
| tenant_id | UUID | NOT NULL, FK → Tenant.id | |
| case_id | UUID | NOT NULL, FK → Case.id | |
| author_id | UUID | NOT NULL, FK → User.id | |
| body | TEXT | NOT NULL | Message content |
| is_internal | BOOLEAN | NOT NULL, DEFAULT false | Internal note vs customer-visible (FR-046, FR-047) |
| communication_type | VARCHAR(20) | NOT NULL, DEFAULT 'reply' | 'reply', 'note', 'system', 'macro' |
| macro_id | UUID | NULLABLE, FK → Macro.id | If applied via macro |
| email_message_id | VARCHAR(255) | NULLABLE | For email-sourced messages |
| email_in_reply_to | VARCHAR(255) | NULLABLE | Threading |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() | |
| updated_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() | |
| edited_at | TIMESTAMPTZ | NULLABLE | If edited |

**Indexes**: `INDEX (tenant_id, case_id, created_at)`, `INDEX (tenant_id, author_id, created_at)`

---

### Attachment
**Purpose**: File attachments on communications/cases (FR-049)

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | |
| tenant_id | UUID | NOT NULL, FK → Tenant.id | |
| case_id | UUID | NOT NULL, FK → Case.id | |
| communication_id | UUID | NULLABLE, FK → Communication.id | Optional link |
| filename | VARCHAR(255) | NOT NULL | Original filename |
| stored_filename | VARCHAR(255) | NOT NULL | S3 object key |
| content_type | VARCHAR(100) | NOT NULL | MIME type |
| size_bytes | BIGINT | NOT NULL | File size |
| checksum | VARCHAR(64) | NOT NULL | SHA-256 for integrity |
| uploaded_by_id | UUID | NOT NULL, FK → User.id | |
| virus_scanned | BOOLEAN | NOT NULL, DEFAULT false | |
| virus_scan_result | VARCHAR(20) | NULLABLE | 'clean', 'infected', 'error' |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() | |

**Indexes**: `INDEX (tenant_id, case_id, created_at)`, `INDEX (stored_filename)`

---

### SLARule
**Purpose**: Response/resolution targets by category/priority/queue (FR-070)

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | |
| tenant_id | UUID | NOT NULL, FK → Tenant.id | |
| name | VARCHAR(100) | NOT NULL | |
| description | TEXT | NULLABLE | |
| category_id | UUID | NULLABLE, FK → Category.id | NULL = all categories |
| priority_id | UUID | NULLABLE, FK → Priority.id | NULL = all priorities |
| queue_id | UUID | NULLABLE, FK → Queue.id | NULL = all queues |
| response_target_minutes | INTEGER | NULLABLE | Business minutes for first response |
| resolution_target_minutes | INTEGER | NULLABLE | Business minutes for resolution |
| business_hours_id | UUID | NULLABLE, FK → BusinessHours.id | Business hours calendar |
| is_active | BOOLEAN | NOT NULL, DEFAULT true | |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() | |
| updated_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() | |

**Indexes**: `INDEX (tenant_id, is_active)`, `INDEX (tenant_id, category_id, priority_id, queue_id)`

**Matching Logic**: Most specific match wins (category+priority+queue > category+priority > category > priority > queue > default)

---

### BusinessHours
**Purpose**: Business hours calendar for SLA calculations

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | |
| tenant_id | UUID | NOT NULL, FK → Tenant.id | |
| name | VARCHAR(100) | NOT NULL | |
| timezone | VARCHAR(50) | NOT NULL | IANA timezone |
| monday_start | TIME | NULLABLE | |
| monday_end | TIME | NULLABLE | |
| tuesday_start | TIME | NULLABLE | |
| tuesday_end | TIME | NULLABLE | |
| wednesday_start | TIME | NULLABLE | |
| wednesday_end | TIME | NULLABLE | |
| thursday_start | TIME | NULLABLE | |
| thursday_end | TIME | NULLABLE | |
| friday_start | TIME | NULLABLE | |
| friday_end | TIME | NULLABLE | |
| saturday_start | TIME | NULLABLE | |
| saturday_end | TIME | NULLABLE | |
| sunday_start | TIME | NULLABLE | |
| sunday_end | TIME | NULLABLE | |
| holidays | JSONB | DEFAULT '[]' | Array of {date, name} |
| is_default | BOOLEAN | NOT NULL, DEFAULT false | One per tenant |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() | |
| updated_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() | |

---

### Macro
**Purpose**: Preconfigured message templates with field updates (FR-051, FR-052)

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | |
| tenant_id | UUID | NOT NULL, FK → Tenant.id | |
| name | VARCHAR(100) | NOT NULL | |
| description | TEXT | NULLABLE | |
| body_template | TEXT | NOT NULL | Message template (supports variables) |
| is_internal | BOOLEAN | NOT NULL, DEFAULT false | Creates internal note if true |
| set_status_id | UUID | NULLABLE, FK → Status.id | Status to apply |
| set_priority_id | UUID | NULLABLE, FK → Priority.id | Priority to apply |
| set_category_id | UUID | NULLABLE, FK → Category.id | Category to apply |
| set_queue_id | UUID | NULLABLE, FK → Queue.id | Queue to apply |
| set_assigned_agent_id | UUID | NULLABLE, FK → User.id | Agent to assign |
| shortcut_key | VARCHAR(20) | NULLABLE, UNIQUE per tenant | Keyboard shortcut (FR-053) |
| is_active | BOOLEAN | NOT NULL, DEFAULT true | |
| created_by_id | UUID | NOT NULL, FK → User.id | |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() | |
| updated_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() | |

**Indexes**: `UNIQUE (tenant_id, shortcut_key) WHERE shortcut_key IS NOT NULL`, `INDEX (tenant_id, is_active)`

---

### AuditLog
**Purpose**: Immutable audit trail (FR-100, FR-101, FR-102, FR-103)

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | |
| tenant_id | UUID | NOT NULL, FK → Tenant.id | |
| event_type | VARCHAR(50) | NOT NULL | Enum: see Event Types below |
| actor_id | UUID | NOT NULL, FK → User.id | Who performed action |
| actor_role | VARCHAR(50) | NOT NULL | Actor's role at time |
| entity_type | VARCHAR(50) | NOT NULL | 'case', 'user', 'queue', 'config', etc. |
| entity_id | UUID | NOT NULL | Target entity ID |
| before | JSONB | NULLABLE | Previous state (relevant fields) |
| after | JSONB | NULLABLE | New state (relevant fields) |
| metadata | JSONB | DEFAULT '{}' | Additional context (IP, user-agent, etc.) |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() | Immutable |

**Indexes**: 
- `INDEX (tenant_id, created_at DESC)`
- `INDEX (tenant_id, entity_type, entity_id, created_at DESC)`
- `INDEX (tenant_id, actor_id, created_at DESC)`
- `INDEX (tenant_id, event_type, created_at DESC)`

**Event Types** (audit-generating actions):
- `case.created`, `case.updated`, `case.status_changed`, `case.assigned`, `case.escalated`, `case.closed`
- `communication.created`, `communication.updated`
- `customer.created`, `customer.updated`
- `user.created`, `user.updated`, `user.role_changed`, `user.deactivated`, `user.reactivated`
- `queue.created`, `queue.updated`, `queue.deactivated`
- `category.created`, `category.updated`, `category.deactivated`
- `status.created`, `status.updated`, `status.deactivated`
- `priority.created`, `priority.updated`, `priority.deactivated`
- `sla_rule.created`, `sla_rule.updated`, `sla_rule.deactivated`
- `macro.created`, `macro.updated`, `macro.deactivated`
- `organization.created`, `organization.updated`, `organization.deactivated`
- `attachment.uploaded`, `attachment.deleted`
- `login.success`, `login.failed`, `logout`, `password.changed`, `mfa.changed`

---

## Supporting Entities

### CaseWatcher
**Purpose**: Users watching a case for real-time notifications

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | |
| tenant_id | UUID | NOT NULL, FK → Tenant.id | |
| case_id | UUID | NOT NULL, FK → Case.id | |
| user_id | UUID | NOT NULL, FK → User.id | |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() | |

**Indexes**: `UNIQUE (case_id, user_id)`, `INDEX (tenant_id, user_id)`

---

### CaseActivity
**Purpose**: Lightweight activity feed (FR-035) — denormalized for performance

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | |
| tenant_id | UUID | NOT NULL, FK → Tenant.id | |
| case_id | UUID | NOT NULL, FK → Case.id | |
| actor_id | UUID | NOT NULL, FK → User.id | |
| activity_type | VARCHAR(50) | NOT NULL | 'status_change', 'assignment', 'comment', 'priority_change', etc. |
| summary | VARCHAR(500) | NOT NULL | Human-readable summary |
| metadata | JSONB | DEFAULT '{}' | Structured details |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() | |

**Indexes**: `INDEX (tenant_id, case_id, created_at DESC)`

---

### EmailIngestionLog
**Purpose**: Track email-to-case processing for debugging/audit (FR-025, FR-026)

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | |
| tenant_id | UUID | NOT NULL, FK → Tenant.id | |
| provider | VARCHAR(50) | NOT NULL | 'postmark' |
| message_id | VARCHAR(255) | NOT NULL | Provider's message ID |
| thread_id | VARCHAR(255) | NULLABLE | Thread identifier |
| from_email | VARCHAR(255) | NOT NULL | |
| to_email | VARCHAR(255) | NOT NULL | |
| subject | VARCHAR(500) | NOT NULL | |
| case_id | UUID | NULLABLE, FK → Case.id | Matched case |
| status | VARCHAR(20) | NOT NULL | 'processed', 'created_case', 'attached_to_case', 'failed', 'duplicate' |
| error_message | TEXT | NULLABLE | If failed |
| raw_payload | JSONB | NOT NULL | Full webhook payload |
| processed_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() | |

**Indexes**: `UNIQUE (tenant_id, provider, message_id)`, `INDEX (tenant_id, case_id)`, `INDEX (tenant_id, status, processed_at)`

---

## Value Objects / Embedded Types

### SLATargets (computed, not stored directly)
```typescript
interface SLATargets {
  responseTargetAt: Date | null;
  resolutionTargetAt: Date | null;
  isPaused: boolean;
  pausedAt: Date | null;
  totalPausedDuration: Duration;
}
```

### CaseRoutingResult (FR-023)
```typescript
interface CaseRoutingResult {
  queueId: UUID;
  statusId: UUID;
  priorityId: UUID;
  categoryId: UUID | null;
  assignedAgentId: UUID | null;
  warnings: string[]; // e.g., "Using catch-all queue: routing rule evaluation failed"
}
```

### EmailParsingResult (FR-026)
```typescript
interface EmailParsingResult {
  caseId: UUID | null;          // Matched existing case
  shouldCreateCase: boolean;    // No match found
  threadId: string | null;
  inReplyTo: string | null;
  references: string[];
  customerEmail: string;
}
```

---

## State Transitions (Case)

```
NEW
  │
  ├──→ OPEN (agent picks up, or auto-assign)
  │
  ├──→ PENDING_CUSTOMER (agent sends response, sets pending_customer)
  │       │
  │       ├──→ OPEN (customer replies — automatic per FR-032)
  │       │
  │       └──→ RESOLVED (if customer confirms resolution via portal)
  │
  ├──→ PENDING_INTERNAL (agent escalates, sets internal owner/dept)
  │       │
  │       └──→ OPEN (internal owner completes work)
  │
  └──→ RESOLVED (agent resolves with root_cause, category, final_priority)
          │
          └──→ CLOSED (auto after N days, or manual close)
```

**Transition Guards**:
- `NEW → OPEN`: Always allowed for agents
- `OPEN → PENDING_CUSTOMER`: Requires outbound communication
- `PENDING_CUSTOMER → OPEN`: Automatic on inbound customer communication
- `OPEN → PENDING_INTERNAL`: Requires internal_owner_id OR internal_department
- `OPEN → RESOLVED`: Requires root_cause, category_id, final_priority_id populated and validated
- `PENDING_INTERNAL → RESOLVED`: Requires root_cause, category_id, final_priority_id
- `RESOLVED → CLOSED`: Allowed after configurable delay (default 7 days) or manual

**SLA Timer Behavior**:
- Running in: NEW, OPEN, PENDING_INTERNAL, RESOLVED
- Paused in: PENDING_CUSTOMER
- N/A in: CLOSED

---

## Validation Rules (Application Layer)

### Case Creation
- Subject: 1-500 chars, required
- Description: 1-10000 chars, required
- Customer: Must exist and be active
- Default status: First system status where `is_system=true` AND `name='new'`
- Default priority: First active priority sorted by sort_order
- Default queue: Routing rule evaluation → catch-all on failure (FR-023)

### Status Transitions
- To RESOLVED: `root_cause` NOT NULL, `category_id` NOT NULL, `final_priority_id` NOT NULL
- To PENDING_INTERNAL: `internal_owner_id` XOR `internal_department` required
- Deactivation of status/category/queue/priority blocked if referenced by active cases

### SLA Calculation
- Business minutes only (per BusinessHours)
- Excludes holidays
- Paused time subtracted from elapsed
- Target = created_at + target_minutes (business) - paused_duration

---

## Seed Data Requirements (NFR-008)

Minimum demo data per tenant:
- 1 Tenant (demo)
- 1 Admin user
- 2 Agent users
- 1 Supervisor user
- 3 Customer users
- 1 Organization
- 6 System Statuses (new, open, pending_customer, pending_internal, resolved, closed)
- 4 Priorities (low, medium, high, critical)
- 5 Categories (Technical, Billing, Account, General, Feature Request)
- 3 Queues (General Support, Technical Support, Billing) + 1 Catch-All
- 3 SLA Rules (Critical: 15min/4hr, High: 1hr/8hr, Default: 4hr/24hr)
- 1 Business Hours (Mon-Fri 9-5, UTC)
- 5 Macros (common responses)
- 20 Cases across statuses/priorities
- Communications, attachments, audit logs for cases