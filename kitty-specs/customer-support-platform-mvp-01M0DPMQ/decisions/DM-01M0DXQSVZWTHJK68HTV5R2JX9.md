# Decision Moment `01M0DXQSVZWTHJK68HTV5R2JX9`

- **Mission:** `customer-support-platform-mvp-01M0DPMQ`
- **Origin flow:** `plan`
- **Slot key:** `plan.technical-context.realtime-mechanism`
- **Input key:** `realtime_mechanism`
- **Status:** `resolved`
- **Created:** `2026-08-19T21:09:51.615282+00:00`
- **Resolved:** `2026-08-19T21:43:02.960418+00:00`
- **Opened by:** `cli`
- **Other answer:** `false`

## Question

Which real-time transport mechanism should be used for live case/queue updates?

## Options

- WebSockets (Socket.io) - bidirectional, lower latency
- Server-Sent Events (SSE) - simpler, HTTP/2 compatible, auto-reconnect
- Hybrid: SSE for updates + REST for mutations
- Defer to implementation

## Final answer

Hybrid: SSE for real-time server-to-client updates + REST for mutations

## Rationale

_(none)_

## Change log

- `2026-08-19T21:09:51.615282+00:00` — opened
- `2026-08-19T21:43:02.960418+00:00` — resolved (final_answer="Hybrid: SSE for real-time server-to-client updates + REST for mutations")
