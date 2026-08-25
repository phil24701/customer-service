---
work_package_id: WP11
title: Email Integration (Postmark)
dependencies:
- WP04
- WP05
requirement_refs:
- C-014
- FR-025
- FR-026
tracker_refs: []
planning_base_branch: epic1
merge_target_branch: epic1
branch_strategy: Planning artifacts for this mission were generated on epic1. During /spec-kitty.implement this WP may branch from a dependency-specific base, but completed changes must merge back into epic1 unless the human explicitly redirects the landing branch.
subtasks:
- T047
- T048
- T049
- T050
agent: claude
history:
- timestamp: '2026-08-25T18:45:19Z'
  action: created
  actor: spec-kitty-tasks
agent_profile: implementer-ivan
authoritative_surface: backend/src/email/
create_intent: []
execution_mode: code_change
model: claude-sonnet-4-6
owned_files:
- backend/src/email/**
- backend/src/webhooks/email/**
role: implementer
tags: []
---

## ⚡ Do This First: Load Agent Profile

```bash
/opencode/skill ad-hoc-profile-load --name implementer-ivan
```

---

## Objective

Implement webhook-based email-to-case ingestion with Postmark, provider-independent boundary, inbound email association, signature verification, dead-letter queue, and outbound notifications.

---

## Context

- **Dependencies**: WP04 (Case, routing), WP05 (Communications, Attachments)
- **Reference Documents**: spec.md (FR-025, FR-026, C-014), plan.md (IC-12), data-model.md (EmailIngestionLog), contracts/openapi.yaml (Webhooks tag), research.md (Email-to-Case Ingestion Rules, Integration Patterns), DM-01M0E1QJ (Postmark), DM-01M0DSFK (webhook-based)

---

## Detailed Guidance per Subtask

### T047: Build email ingestion webhook endpoint (Postmark adapter)

**Purpose**: Webhook receiver per FR-025, C-014.

**Steps**:
1. Create `src/email/adapters/postmark.adapter.ts` implementing `EmailProvider`:
   - `verifyWebhookSignature(payload, signature): boolean` - Postmark uses HMAC-SHA256
   - `parseInboundWebhook(payload): InboundEmail` - extract from, to, subject, body, attachments, Message-ID, In-Reply-To, References
2. Create `src/webhooks/email/controllers/email-webhook.controller.ts`:
   - `POST /webhooks/email/postmark` - raw body for signature verification
   - Validate signature, parse, process
3. Create `src/email/services/email-ingestion.service.ts`:
   - `processInboundEmail(email: InboundEmail): Promise<ProcessingResult>`
   - Log to EmailIngestionLog (status: processed/created_case/attached_to_case/failed/duplicate)

**Files**: `backend/src/email/adapters/postmark.adapter.ts`, `backend/src/webhooks/email/controllers/`, `backend/src/email/services/email-ingestion.service.ts`

**Validation**:
- [ ] Webhook signature verified
- [ ] Inbound email parsed correctly
- [ ] EmailIngestionLog created for each
- [ ] Raw payload stored for debugging

---

### T048: Implement inbound email parsing & case association (threading)

**Purpose**: Email threading per FR-026.

**Steps**:
1. Create `src/email/services/email-threading.service.ts`:
   - `findOrCreateCase(email: InboundEmail): Promise<EmailParsingResult>`
   - Match by: In-Reply-To → References → Message-ID → subject prefix (CSP-XXXXXX)
   - If match found: attach as communication to existing case
   - If no match: create new case (customer from from_email)
   - Handle: from_email not in system → create Customer record
2. Create communication with: is_internal=false, communication_type='reply', email_message_id, email_in_reply_to
3. If case was pending_customer → transition to open (WP06/T029)
4. Update case email_thread_id for future matching

**Files**: `backend/src/email/services/email-threading.service.ts`, `backend/src/email/services/email-ingestion.service.ts`

**Validation**:
- [ ] Reply to existing case attaches correctly
- [ ] New email from unknown customer creates case + customer
- [ ] Threading via Message-ID/References works
- [ ] Case auto-opens from pending_customer

---

### T049: Add webhook signature verification and dead-letter queue for failures [P]

**Purpose**: Security and reliability per research.md.

**Steps**:
1. Signature verification in PostmarkAdapter:
   - Postmark: `X-Postmark-Signature` header, HMAC-SHA256 with webhook secret
   - Reject invalid with 401
2. Dead-letter handling:
   - On any processing failure: log to EmailIngestionLog (status=failed, error_message)
   - Add to BullMQ `email-dead-letter` queue for retry/admin review
   - Admin endpoint: `GET /admin/email/dead-letter`, `POST /admin/email/dead-letter/:id/retry`
3. Rate limiting: 100 requests/minute per IP on webhook endpoint

**Files**: `backend/src/email/adapters/postmark.adapter.ts`, `backend/src/email/services/dead-letter.service.ts`, `backend/src/admin/controllers/email-dead-letter.controller.ts`

**Validation**:
- [ ] Invalid signature → 401
- [ ] Failed processing → dead-letter queue
- [ ] Admin can view/retry dead letters
- [ ] Rate limiting active

---

### T050: Create outbound email notification service (provider-independent) [P]

**Purpose**: Outbound notifications per C-014.

**Steps**:
1. Create `src/email/services/email-outbound.service.ts`:
   - `sendNotification(to, subject, body, caseId?): Promise<SendResult>`
   - Uses EmailProvider interface (PostmarkAdapter)
   - Template support for common notifications (case created, assigned, updated, resolved)
3. Integrate with case operations: on assignment, status change, new communication
4. Queue via BullMQ `email-delivery` (WP12) for async sending
5. Track in EmailIngestionLog (outbound) or new NotificationLog

**Files**: `backend/src/email/services/email-outbound.service.ts`, `backend/src/email/email.module.ts`

**Validation**:
- [ ] Outbound emails sent via Postmark
- [ ] Provider-independent (swap adapter)
- [ ] Async via BullMQ
- [ ] Notifications on key events

---

## Branch Strategy

- **Planning Branch**: `epic1`
- **Work Package Branch**: `feature/wp11-email-integration` (from `epic1`)
- **Final Merge Target**: `epic1`
- **Implementation Command**: `spec-kitty agent action implement WP11 --agent claude`

---

## Test Strategy

- Unit tests for Postmark signature verification
- Unit tests for threading logic (match, no match, new customer)
- Integration test for webhook endpoint
- Test dead-letter retry

---

## Definition of Done

- [ ] Postmark webhook receives and verifies inbound email
- [ ] Threading: replies attach to existing case
- [ ] New emails create cases with customer
- [ ] Signature verification + dead-letter queue
- [ ] Outbound notifications async via BullMQ
- [ ] Tests pass
- [ ] Swagger matches contracts

---

## Risks

- Webhook signature verification - must match Postmark exactly
- Email parsing reliability - handle malformed emails
- Provider API changes - adapter pattern isolates
- Bounce/complaint handling - Postmark webhook for bounces
- Rate limiting - protect webhook endpoint

---

## Reviewer Guidance

- Verify HMAC-SHA256 signature verification matches Postmark docs
- Test threading: In-Reply-To → References → subject prefix
- Confirm dead-letter captures all failure modes
- Check adapter pattern allows provider swap