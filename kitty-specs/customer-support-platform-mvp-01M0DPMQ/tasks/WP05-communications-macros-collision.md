---
work_package_id: WP05
title: Communications, Attachments, Macros & Collision Prevention
dependencies:
- WP04
requirement_refs:
- FR-039
- FR-040
- FR-045
- FR-046
- FR-047
- FR-048
- FR-049
- FR-050
- FR-051
- FR-052
- FR-053
tracker_refs: []
planning_base_branch: epic1
merge_target_branch: epic1
branch_strategy: Planning artifacts for this mission were generated on epic1. During /spec-kitty.implement this WP may branch from a dependency-specific base, but completed changes must merge back into epic1 unless the human explicitly redirects the landing branch.
subtasks:
- T023
- T024
- T025
- T026
- T027
agent: claude
history:
- timestamp: '2026-08-25T18:45:19Z'
  action: created
  actor: spec-kitty-tasks
agent_profile: implementer-ivan
authoritative_surface: backend/src/communications/
create_intent: []
execution_mode: code_change
model: claude-sonnet-4-6
owned_files:
- backend/src/communications/**
- backend/src/attachments/**
- backend/src/macros/**
- backend/src/collision/**
role: implementer
tags: []
---

## ⚡ Do This First: Load Agent Profile

Before reading any other context, load your assigned agent profile:

```bash
/opencode/skill ad-hoc-profile-load --name implementer-ivan
```

This will apply the implementer persona with its identity, governance scope, boundaries, and initialization declarations. Do not proceed until the profile is loaded.

---

## Objective

Implement customer-visible messages vs internal notes, file attachments via S3/MinIO signed URLs, macro system (templates + field updates + keyboard shortcuts), and real-time collision prevention with drafting locks.

---

## Context

- **Mission**: customer-support-platform-mvp-01M0DPMQ
- **Dependencies**: WP04 (Case lifecycle working)
- **Reference Documents**: spec.md (FR-045 through FR-053, FR-039, FR-040), plan.md (IC-05, IC-12, IC-14), data-model.md (Communication, Attachment, Macro), contracts/openapi.yaml (Communications, Attachments, Macros tags), research.md (Real-Time Collision Prevention, Email-to-Case Ingestion Rules, Integration Patterns)

---

## Detailed Guidance per Subtask

### T023: Create Communication, Attachment, Macro, AuditLog, EmailIngestionLog models

**Purpose**: Database models (extends WP01/T002).

**Steps**:
1. Verify Prisma schema includes Communication, Attachment, Macro, EmailIngestionLog per data-model.md
2. Communication: is_internal (customer-visible vs internal note), communication_type (reply, note, system, macro), macro_id
3. Attachment: stored_filename (S3 key), checksum (SHA-256), virus_scanned, virus_scan_result
4. Macro: body_template (with variable substitution), set_* fields for field updates, shortcut_key (unique per tenant)
5. EmailIngestionLog: for debugging email-to-case (WP11)
6. Create migration: `npx prisma migrate dev --name communications-macros`
7. Create repositories for each

**Files**: `backend/prisma/schema.prisma`, `backend/prisma/migrations/`, `backend/src/communications/repositories/`, `backend/src/attachments/repositories/`, `backend/src/macros/repositories/`

**Validation**:
- [ ] Migration applies cleanly
- [ ] Macro shortcut_key unique per tenant
- [ ] Attachment checksum stored
- [ ] Communication types enum correct

---

### T024: Build communications endpoints (reply, internal note, chronological list)

**Purpose**: REST API per FR-045 through FR-050, contracts/openapi.yaml Communications tag.

**Steps**:
1. Create DTOs: `CreateCommunicationDto` (body, isInternal, macroId, attachmentIds)
2. Create `src/communications/controllers/communications.controller.ts`:
   - `GET /cases/:id/communications` - paginated chronological list
   - `POST /cases/:id/communications` - add reply or internal note
   - Filter: customer role sees only !is_internal
3. Create `src/communications/services/communications.service.ts`:
   - `addCommunication(caseId, authorId, dto): Promise<Communication>`
   - Macro expansion if macroId provided
   - If isInternal=false and case in pending_customer → transition to open (FR-032)
   - Create AuditLog, CaseActivity
   - Emit SSE event (case.updated, communication.created)
4. Variable substitution in macros: {{case_number}}, {{customer_name}}, {{agent_name}}, {{status}}, {{priority}}

**Files**: `backend/src/communications/dto/`, `backend/src/communications/controllers/`, `backend/src/communications/services/`, `backend/src/communications/communications.module.ts`

**Validation**:
- [ ] Customer sees only customer-visible messages
- [ ] Agent sees all (internal + customer)
- [ ] Chronological order by created_at
- [ ] Macro expands variables correctly
- [ ] Inbound customer communication auto-transitions pending_customer → open

---

### T025: Implement attachment upload via S3/MinIO signed URLs [P]

**Purpose**: File attachments per FR-049, C-014, DM-01M0GKP5.

**Steps**:
1. Create `src/attachments/services/storage.service.ts` (StorageProvider interface):
   - `putObject(key, body, contentType): Promise<PutResult>`
   - `getSignedUrl(key, expiresIn): Promise<string>`
   - `deleteObject(key): Promise<void>`
   - MinIO implementation for local, S3 for prod
2. Create `src/attachments/controllers/attachments.controller.ts`:
   - `POST /cases/:id/attachments` - initiate upload (returns uploadUrl, expiresAt)
   - `GET /attachments/:id` - get signed download URL
   - `DELETE /attachments/:id` - delete (soft delete or hard)
3. Validation: max 50MB, allowed MIME types, virus scan hook (placeholder)
4. On upload complete: client calls PUT to uploadUrl, then POST to confirm → create Attachment record
5. Signed URLs expire in 15 min (upload), 1 hour (download)

**Files**: `backend/src/attachments/services/storage.service.ts`, `backend/src/attachments/controllers/`, `backend/src/attachments/services/attachments.service.ts`

**Validation**:
- [ ] Upload URL generated, client can PUT file to MinIO
- [ ] Download URL works, expires correctly
- [ ] File size/type validation enforced
- [ ] Attachment linked to case/communication
- [ ] Checksum verified on download

---

### T026: Create macro system: templates with field updates, keyboard shortcuts

**Purpose**: Macros per FR-051, FR-052, FR-053.

**Steps**:
1. Create `src/macros/services/macro.service.ts`:
   - `applyMacro(macroId, caseId, authorId): Promise<{ communication: Communication; caseUpdates: Partial<Case> }>`
   - Expand body_template with variables
   - Apply set_status_id, set_priority_id, set_category_id, set_queue_id, set_assigned_agent_id
   - Validate transitions (e.g., can't set resolved without root_cause)
   - Create communication with communication_type='macro', macro_id
   - Create audit log for case field changes
2. Create `src/macros/controllers/macros.controller.ts`:
   - CRUD for macros (admin only for create/update/delete)
   - `GET /macros` - list active macros for user's role
   - `POST /macros/:id/apply` - apply macro to case
3. Keyboard shortcuts: store shortcut_key, frontend handles key binding (WP16)

**Files**: `backend/src/macros/services/macro.service.ts`, `backend/src/macros/controllers/`, `backend/src/macros/macros.module.ts`

**Validation**:
- [ ] Macro creates communication with expanded template
- [ ] Macro updates case fields (status, priority, category, queue, assignment)
- [ ] Field updates validated (transition guards)
- [ ] Shortcut keys unique per tenant
- [ ] Audit log for macro application + field changes

---

### T027: Add collision prevention: drafting lock acquire/release via SSE

**Purpose**: Real-time collision prevention per FR-039, research.md.

**Steps**:
1. Create `src/collision/services/collision.service.ts`:
   - `acquireLock(caseId, userId, type: 'reply' | 'note'): Promise<{ lockExpiresAt: Date }>`
   - `releaseLock(caseId, userId): Promise<void>`
   - `getLock(caseId): Promise<CollisionLock | null>`
   - Lock TTL: 5 minutes, renewable
   - Store in Redis (key: `collision:{caseId}`)
2. Create `src/collision/controllers/collision.controller.ts`:
   - `POST /cases/:id/collision` - acquire lock (returns lockExpiresAt)
   - `DELETE /cases/:id/collision` - release lock
   - 409 if lock held by another user (return CollisionWarning)
3. SSE integration (WP13): emit `collision.warning` when lock acquired, `collision.released` when released
4. Auto-release on: communication submitted, navigation away (beacon API), timeout, connection loss
5. Frontend integration (WP16): show warning, disable editor when locked

**Files**: `backend/src/collision/services/collision.service.ts`, `backend/src/collision/controllers/`, `backend/src/collision/collision.module.ts`

**Validation**:
- [ ] Lock acquired for reply/note type
- [ ] Second user gets 409 with drafting user info
- [ ] Lock auto-expires after 5 min
- [ ] Lock released on submit, navigation, timeout
- [ ] SSE events emitted for warning/release
- [ ] Graceful degradation: if Redis down, fall back to optimistic locking (WP13)

---

## Branch Strategy

- **Planning Branch**: `epic1`
- **Work Package Branch**: `feature/wp05-comms-macros-collision` (from `epic1`)
- **Final Merge Target**: `epic1`
- **Execution Worktree**: Allocated by `finalize-tasks`
- **Implementation Command**: `spec-kitty agent action implement WP05 --agent claude`

---

## Test Strategy

- Unit tests for macro variable expansion and field updates
- Unit tests for collision lock acquire/release/timeout
- Integration tests for communications endpoints
- Integration tests for attachment signed URLs
- Test macro field update validation (transition guards)
- Test SSE collision events

---

## Definition of Done

- [ ] All 5 subtasks complete
- [ ] Communications: reply vs internal note, chronological, macro expansion
- [ ] Attachments: signed upload/download URLs via MinIO/S3
- [ ] Macros: templates + field updates + shortcuts
- [ ] Collision prevention: lock acquire/release, SSE events, auto-release
- [ ] Tests pass (>80% coverage)
- [ ] Swagger matches contracts/openapi.yaml

---

## Risks

- Attachment storage integration (S3/MinIO) - test with both local and prod configs
- Macro field-update side effects - validate through state machine
- Keyboard shortcut conflicts - frontend handles, backend just stores shortcut_key
- Message ordering with real-time updates - use created_at for sorting
- Collision race conditions - Redis atomic operations, TTL

---

## Reviewer Guidance

- Verify customer cannot see isInternal=true communications
- Check macro variable expansion covers all specified variables
- Confirm attachment checksum verification
- Validate collision lock TTL and auto-release scenarios
- Ensure SSE events match contracts/openapi.yaml SSEEvent schema