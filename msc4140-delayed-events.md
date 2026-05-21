# MSC4140: Cancellable Delayed Events

## Summary

MSC4140 introduces a mechanism for Matrix clients to **schedule an event to be sent at a later time**. The homeserver is responsible for sending the event even if the client is offline or disconnected. This enables a "deadman switch" pattern for MatrixRTC: a client schedules a leave event before joining, then periodically resets the timer. If the client crashes or loses connectivity, the homeserver automatically sends the leave event.

---

## ⚡ Live Update (May 21, 2026)

MSC4140 is now **"Proposed for FCP readiness"** in the SCT project — the most significant status change since this research began.

- @turt2live on May 19, 2026: *"this is looking closer to being ready for FCP"*
- @Johennes conducted a comprehensive review pass May 18–20, 2026 with wording suggestions
- **MSC4479** ("Shorten alternatives") was **merged** on May 20, 2026
- **MSC4478** ("Move use cases section out") is in progress (May 20, 2026)
- Implementation TODOs removed from PR description (May 20, 2026) — tracked in comments instead
- Labels updated: now includes `00-weekly-pings` (weekly SCT tracking) alongside `implementation-needs-checking`

## Status

| Field | Value |
|-------|-------|
| **PR** | [#4140](https://github.com/matrix-org/matrix-spec-proposals/pull/4140) |
| **State** | Open |
| **Author** | [@toger5](https://github.com/toger5) (Timo K, Element) |
| **Created** | May 7, 2024 |
| **Last updated** | May 20, 2026 |
| **Labels** | `00-weekly-pings`, `voip`, `proposal`, `client-server`, `kind:feature`, `matrix-2.0`, `implementation-needs-checking` |
| **FCP** | **Proposed for FCP readiness** — active SCT review; @turt2live requested changes; @Johennes actively reviewing |
| **SCT Project Status** | "Proposed for FCP readiness" |
| **Lifecycle stage** | Final spec review pass; FCP expected Q2/Q3 2026 |

### Implementations

| Component | Status | Link |
|-----------|--------|------|
| Synapse | ✅ Done (5 PRs) | [#17326](https://github.com/element-hq/synapse/pull/17326), [#19038](https://github.com/element-hq/synapse/pull/19038), [#19354](https://github.com/element-hq/synapse/pull/19354), [#19479](https://github.com/element-hq/synapse/pull/19479), [#19539](https://github.com/element-hq/synapse/pull/19539) |
| matrix-js-sdk | ✅ Done (3 PRs) | [#4294](https://github.com/matrix-org/matrix-js-sdk/pull/4294), [#5066](https://github.com/matrix-org/matrix-js-sdk/pull/5066), [#5133](https://github.com/matrix-org/matrix-js-sdk/pull/5133) |
| Element Call (SPA) | ✅ Done | [element-hq/element-call#2529](https://github.com/element-hq/element-call/pull/2529) |
| Element Web (embedded) | ✅ Done | [matrix-org/matrix-widget-api#90](https://github.com/matrix-org/matrix-widget-api/pull/90), [#143](https://github.com/matrix-org/matrix-widget-api/pull/143) |
| Element X | ✅ Done | [ruma/ruma#1845](https://github.com/ruma/ruma/pull/1845), [matrix-org/matrix-rust-sdk#3600](https://github.com/matrix-org/matrix-rust-sdk/pull/3600) |

### Open spec issues (not implementation blockers)

- `M_INVALID_PARAM` vs `M_MAX_DELAY_EXCEEDED` error code — likely moving max delay to `/capabilities` instead, making the custom error unnecessary.
- `running_since` field naming — likely rename to `scheduled_ts` for consistency with other `..._ts` fields.
- `delay_id` vs `delay_token` — @Johennes proposed rename for security clarity; @AndrewFerr against (OAuth scopes are the long-term answer).
- Use cases section to be shortened or removed (MSC4478 in progress).

---

## Proposal

Five new Client-Server API operations are introduced:

### 1. Schedule a delayed event

```
PUT /_matrix/client/v3/rooms/{roomId}/delayed_event/{eventType}/{txnId}
```

Body:
- `delay` (required) — milliseconds to wait before sending.
- `state_key` (optional) — if it's a state event.
- `content` (required) — event content.

Response: `{ "delay_id": "1234567890" }`

The server may add up to 30 seconds to the scheduled time for batch efficiency.

### 2. Get list of delayed events

```
GET /_matrix/client/v3/delayed_events
```

Returns all scheduled events for the current user.

### 3. Restart the timer

```
POST /_matrix/client/v3/delayed_events/{delay_id}
Body: { "action": "restart" }
```

Resets the countdown to the original delay. Used as the "heartbeat" to keep a MatrixRTC membership alive.

### 4. Send immediately

```
POST /_matrix/client/v3/delayed_events/{delay_id}
Body: { "action": "send" }
```

Sends the event now, before the timer expires.

### 5. Cancel

```
POST /_matrix/client/v3/delayed_events/{delay_id}
Body: { "action": "cancel" }
```

Removes the scheduled event without sending it.

---

## MatrixRTC Use Case (Deadman Switch)

```
Client joins call
  │
  ├─► Schedule delayed leave event (delay: 15–30 seconds)
  │     → server holds leave event
  │
  └─► Loop while connected:
        │  periodic heartbeat (restart timer) ──────► server resets countdown
        │
        If client crashes or loses network:
          timer expires → server sends leave event automatically
        
        If client gracefully leaves:
          send delayed event immediately, or cancel it and send leave manually
```

This pattern makes explicit `m.rtc.member` disconnect events reliable even in crash scenarios.

---

## Companion MSC

- [MSC4309](https://github.com/matrix-org/matrix-spec-proposals/pull/4309): Finalised delayed event on sync — a maintenance MSC that surfaces when a delayed event fires via the sync response.

---

## Supersedes

Potentially supersedes [MSC2228](https://github.com/matrix-org/matrix-spec-proposals/pull/2228) (self-destructing events).

---

## References

- PR: https://github.com/matrix-org/matrix-spec-proposals/pull/4140
- Rendered proposal: https://github.com/matrix-org/matrix-spec-proposals/blob/toger5/expiring-events-keep-alive/proposals/4140-delayed-events-futures.md
