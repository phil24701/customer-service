# Decision Moment `01M0E1QJM5DRB1CV9NQCBN78B9`

- **Mission:** `customer-support-platform-mvp-01M0DPMQ`
- **Origin flow:** `plan`
- **Slot key:** `plan.technical-context.email-provider`
- **Input key:** `email_provider`
- **Status:** `resolved`
- **Created:** `2026-08-19T22:19:38.501414+00:00`
- **Resolved:** `2026-08-20T21:41:58.961808+00:00`
- **Opened by:** `cli`
- **Other answer:** `false`

## Question

Which email provider should be used for email-to-case ingestion (FR-025) and outbound notifications?

## Options

- SendGrid (popular, good deliverability, webhook support)
- Mailgun (developer-friendly, good API)
- Postmark (transactional focus, high deliverability)
- Amazon SES (cost-effective, AWS integration)
- Resend (modern, React email support)
- Defer to implementation

## Final answer

Postmark (free Developer plan for portfolio demo, isolated behind provider-independent boundary)

## Rationale

_(none)_

## Change log

- `2026-08-19T22:19:38.501414+00:00` — opened
- `2026-08-20T21:41:58.961808+00:00` — resolved (final_answer="Postmark (free Developer plan for portfolio demo, isolated behind provider-independent boundary)")
