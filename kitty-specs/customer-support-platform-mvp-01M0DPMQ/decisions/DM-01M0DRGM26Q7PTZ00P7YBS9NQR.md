# Decision Moment `01M0DRGM26Q7PTZ00P7YBS9NQR`

- **Mission:** `customer-support-platform-mvp-01M0DPMQ`
- **Origin flow:** `specify`
- **Slot key:** `specify.realtime.mechanism`
- **Input key:** `realtime_mechanism`
- **Status:** `resolved`
- **Created:** `2026-08-19T19:38:33.414355+00:00`
- **Resolved:** `2026-08-19T19:40:06.721951+00:00`
- **Opened by:** `cli`
- **Other answer:** `false`

## Question

What real-time mechanism should be used for live case/queue updates and agent collision prevention? Options: (A) WebSockets (bidirectional, lower latency, more complex), (B) Server-Sent Events - SSE (simpler, unidirectional, HTTP/2 friendly), (C) WebSockets for collision prevention + SSE for queue updates (hybrid), (D) Decide during technical design with tradeoffs documented

## Options

- WebSockets
- Server-Sent Events (SSE)
- Hybrid (WebSockets + SSE)
- Decide during technical design

## Final answer

Use an event-driven application model for real-time behavior (collision prevention and case/queue updates). Defer the specific transport choice (WebSockets, SSE, or hybrid) to technical design based on actual communication patterns and reliability requirements. Document tradeoffs in an architecture decision record.

## Rationale

_(none)_

## Change log

- `2026-08-19T19:38:33.414355+00:00` — opened
- `2026-08-19T19:40:06.721951+00:00` — resolved (final_answer="Use an event-driven application model for real-time behavior (collision prevention and case/queue updates). Defer the specific transport choice (WebSockets, SSE, or hybrid) to technical design based on actual communication patterns and reliability requirements. Document tradeoffs in an architecture decision record.")
