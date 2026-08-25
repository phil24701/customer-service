# Decision Moment `01M0DRN9JAET99K8VMGQ5CT3SM`

- **Mission:** `customer-support-platform-mvp-01M0DPMQ`
- **Origin flow:** `specify`
- **Slot key:** `specify.async.mechanism`
- **Input key:** `async_mechanism`
- **Status:** `resolved`
- **Created:** `2026-08-19T19:41:06.506497+00:00`
- **Resolved:** `2026-08-19T19:55:22.306979+00:00`
- **Opened by:** `cli`
- **Other answer:** `false`

## Question

What background job mechanism should handle asynchronous processing (email-to-case ingestion, SLA timer management, notifications, audit logging)? Options: (A) Bull/BullMQ with Redis (popular, robust, good UI), (B) Node.js worker_threads with custom queue (no extra dependency, simpler ops), (C) PostgreSQL-based queue (pg-boss, uses existing DB, transactional), (D) Decide during technical design with tradeoffs documented

## Options

- Bull/BullMQ with Redis
- Node.js worker_threads custom queue
- PostgreSQL-based queue (pg-boss)
- Decide during technical design

## Final answer

Defer the background job mechanism choice (Bull/BullMQ, worker_threads, pg-boss, or alternative) to technical design. Document tradeoffs in an architecture decision record considering: operational complexity, existing infrastructure (Redis already planned for caching), transactional requirements, and portfolio demonstration value.

## Rationale

_(none)_

## Change log

- `2026-08-19T19:41:06.506497+00:00` — opened
- `2026-08-19T19:55:22.306979+00:00` — resolved (final_answer="Defer the background job mechanism choice (Bull/BullMQ, worker_threads, pg-boss, or alternative) to technical design. Document tradeoffs in an architecture decision record considering: operational complexity, existing infrastructure (Redis already planned for caching), transactional requirements, and portfolio demonstration value.")
