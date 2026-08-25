# Decision Moment `01M0DSFKFX2CNDG6HGJDD2PVNN`

- **Mission:** `customer-support-platform-mvp-01M0DPMQ`
- **Origin flow:** `specify`
- **Slot key:** `specify.email.integration`
- **Input key:** `email_integration`
- **Status:** `resolved`
- **Created:** `2026-08-19T19:55:28.637823+00:00`
- **Resolved:** `2026-08-19T20:03:32.660398+00:00`
- **Opened by:** `cli`
- **Other answer:** `false`

## Question

How should email-to-case ingestion (FR-025, FR-026) be implemented for the MVP? Options: (A) Polling IMAP/SMTP mailbox with a background job (simpler, no external deps), (B) Webhook-based (e.g., SendGrid, Mailgun, Postmark inbound webhooks - more realistic for production), (C) Both: webhook primary with IMAP fallback for demo flexibility, (D) Decide during technical design with tradeoffs documented

## Options

- IMAP/SMTP polling
- Webhook-based (SendGrid/Mailgun/Postmark)
- Hybrid: webhook primary + IMAP fallback
- Decide during technical design

## Final answer

Webhook-based inbound email integration with a provider-independent abstraction boundary. The specific email provider (SendGrid, Mailgun, Postmark, or other) will be selected during technical design based on cost, capabilities, and ease of demonstration. The application defines an inbound email webhook contract that any provider can implement.

## Rationale

_(none)_

## Change log

- `2026-08-19T19:55:28.637823+00:00` — opened
- `2026-08-19T20:03:32.660398+00:00` — resolved (final_answer="Webhook-based inbound email integration with a provider-independent abstraction boundary. The specific email provider (SendGrid, Mailgun, Postmark, or other) will be selected during technical design based on cost, capabilities, and ease of demonstration. The application defines an inbound email webhook contract that any provider can implement.")
