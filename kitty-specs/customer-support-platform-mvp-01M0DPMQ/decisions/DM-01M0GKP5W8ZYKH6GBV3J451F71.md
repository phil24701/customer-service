# Decision Moment `01M0GKP5W8ZYKH6GBV3J451F71`

- **Mission:** `customer-support-platform-mvp-01M0DPMQ`
- **Origin flow:** `plan`
- **Slot key:** `plan.technical-context.attachment-storage`
- **Input key:** `attachment_storage`
- **Status:** `resolved`
- **Created:** `2026-08-20T22:11:55.912654+00:00`
- **Resolved:** `2026-08-20T22:16:46.002812+00:00`
- **Opened by:** `cli`
- **Other answer:** `false`

## Question

Attachment Storage (from Assumptions): Strategy for file attachments on cases

## Options

- Local filesystem (demo)
- S3-compatible (MinIO for demo, AWS S3 for prod)
- Database BLOB (PostgreSQL bytea)
- Other

## Final answer

S3-compatible (MinIO for local development/demo, configurable endpoint for production S3-compatible provider)

## Rationale

_(none)_

## Change log

- `2026-08-20T22:11:55.912654+00:00` — opened
- `2026-08-20T22:16:46.002812+00:00` — resolved (final_answer="S3-compatible (MinIO for local development/demo, configurable endpoint for production S3-compatible provider)")
