# MSC4196: MatrixRTC Voice and Video Calling Application (`m.call`)

## Summary

MSC4196 defines the **`m.call` application type** — the first first-class MatrixRTC application, specifying how voice and video conferencing works on top of the MSC4143 framework. It defines the specific metadata, slot model, signalling, and UX for calling, including audio/video intent, call ringing integration, and structured disconnect reasons.

*The January 12, 2026 rewrite by @fkwp resolved the open to-dos around voice/video distinction and field structures. The full proposal text was retrieved in May 2026.*

---

## Status

| Field | Value |
|-------|-------|
| **PR** | [#4196](https://github.com/matrix-org/matrix-spec-proposals/pull/4196) |
| **State** | Open (Draft) |
| **Original Author** | [@hughns](https://github.com/hughns) (Hugh Nimmo-Smith, Element) |
| **Rewrite Author** | [@fkwp](https://github.com/fkwp) (Florian Kaltenberger, Element) — January 12, 2026 |
| **Created** | September 19, 2024 |
| **Last updated** | January 12, 2026 (major rewrite) |
| **Labels** | `client-server`, `kind:core`, `matrix-2.0`, `needs-implementation`, `proposal`, `voip` |
| **FCP** | Not started |
| **SCT Project Status** | "Tracking for review" |
| **Lifecycle stage** | Draft; depends on MSC4143 and MSC4075 |
| **Participants** | @hughns, @turt2live, @ara4n, @fkwp |

---

## Slot Model

`m.call` defines a single slot type:

### `m.call#ROOM` — Room-level call slot

- Scoped to a Matrix room, shared among all members
- Managed by a user with sufficient power level
- Suitable for DMs (telephone-style semantics) and group calls
- The `trusted_private_chat` preset SHOULD auto-enable a default `m.call#ROOM` slot

---

## `m.rtc.member` Fields for `m.call`

A valid `m.rtc.member` event with `application.type = "m.call"`:

```json
{
  "slot_id": "m.call#ROOM",
  "sticky_key": "xyzABCDEF0123",
  "member": {
    "id": "xyzABCDEF0123",
    "claimed_device_id": "DEVICEID",
    "claimed_user_id": "@user:matrix.domain"
  },
  "application": {
    "type": "m.call",
    "m.call.id": "some-uuid",
    "m.call.intent": "voice | video",
    "scope": "m.room"
  },
  "m.relates_to": {
    "rel_type": "m.reference",
    "event_id": "$connect_event_id"
  },
  "rtc_transports": [{ "..." : "..." }],
  "versions": ["v0"]
}
```

### Application fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | string | ✅ | Must be `"m.call"` |
| `m.call.id` | string | ⚪ | Optional call instance UUID |
| `m.call.intent` | string | ⚪ | `"voice"` / `"video"` — informational hint only, non-authoritative |
| `scope` | string | ⚪ | `"m.room"` — indicates room-level scope |

### `m.call.intent` semantics

- `"voice"` → clients SHOULD NOT submit a video track initially (but users may upgrade)
- `"video"` → both audio and video tracks expected
- Unknown value → default to `"video"` (both tracks)
- **Non-authoritative** — because of Matrix decentralisation, this is purely a hint

---

## Disconnect Reasons

The `disconnect_reason` field on leave events (`m.rtc.member` with empty content) provides structured error context:

| Class | Example reason | Description |
|-------|---------------|-------------|
| `user_action` | `hangup` | Participant intentionally ended the call |
| `user_action` | `switch_device` | User moved the session to another device |
| `client_error` | `media_error` | Failed to capture/transmit audio/video |
| `client_error` | `transport_failure` | Local ICE/DTLS setup failed |
| `client_error` | `encryption_error` | E2EE setup failure |
| `server_error` | `ice_failed` | ICE negotiation could not complete |
| `server_error` | `dtls_failed` | DTLS handshake failed |
| `server_error` | `network_error` | Temporary network outage |
| `redirection` | `call_transferred` | Call redirected to another slot/device/user |
| `permanent_failure` | `codec_mismatch` | Media codec incompatibility |
| `permanent_failure` | `unsupported_features` | Session requested unsupported capabilities |

---

## Call Ringing for `m.call`

Extends `m.rtc.notification` (MSC4075) with `m.call.intent`:

```json
{
  "type": "m.rtc.notification",
  "content": {
    "sender_ts": 1752583130365,
    "lifetime": 30000,
    "m.mentions": { "user_ids": [], "room": true },
    "m.relates_to": { "rel_type": "m.reference", "event_id": "$rtc_member_event_id" },
    "notification_type": "ring | notification",
    "m.call.intent": "voice | video"
  }
}
```

---

## Open Issues

1. **`needs-implementation` label** — no qualifying implementation exists yet for the revised January 2026 format. Cinny referenced this MSC (Feb 2026) but implementations are based on MSC4143 directly.
2. **`m.call.intent` scope** — already implemented in js-sdk, rust-sdk, and Ruma under various names (`call_intent`, `m.call.intent`). The field naming is not yet finalised across all implementations.

---

## Dependencies

```
MSC4143 (MatrixRTC framework)
    └── MSC4196 (this MSC — m.call application)
            └── MSC4075 (m.rtc.notification — ringing)
                    └── MSC4310 (m.rtc.decline)
```

---

## Unstable Prefix

The `m.call` application type is already embedded within the broader `org.matrix.msc4143.rtc.member` unstable prefix. No additional unstable prefix needed for `m.call` itself.

---

## References

- PR: https://github.com/matrix-org/matrix-spec-proposals/pull/4196
- Raw proposal text: https://raw.githubusercontent.com/matrix-org/matrix-spec-proposals/refs/pull/4196/head/proposals/4196-matrixrtc-m-call.md
- Cinny implementation referencing this MSC: https://github.com/cinnyapp/cinny/pull/2599
