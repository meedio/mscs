# MSC4143: MatrixRTC

## Summary

MSC4143 defines **MatrixRTC** — the foundational protocol for real-time communication (RTC) over Matrix. It replaces the legacy 1:1 VoIP module and the earlier MSC3401 full-mesh approach with a modular, SFU-capable, multi-device framework that scales from ephemeral 1:1 calls to always-on collaborative spaces.

---

## Status

| Field | Value |
|-------|-------|
| **PR** | [#4143](https://github.com/matrix-org/matrix-spec-proposals/pull/4143) |
| **State** | Open |
| **Author** | [@toger5](https://github.com/toger5) (Timo K, Element) |
| **Created** | May 10, 2024 |
| **Last updated** | March 17, 2026 |
| **Labels** | `proposal`, `voip`, `client-server`, `matrix-2.0` |
| **FCP** | Not started |
| **Lifecycle stage** | Community review / implementation testing |

### Blocking dependencies

MSC4143 is **explicitly blocked** on:
1. **MSC4354 (Sticky Events)** — must land and be stable first.
2. **MSC4140 (Cancellable Delayed Events)** — must land and be stable first.

After those land, implementation testing must pass before FCP can be proposed.

---

## Core Concepts

### Applications

An **application type** defines the semantics of an RTC session (e.g. `m.call` for voice/video, or a custom type for games, shared whiteboards). Each application type has its own MSC.

```json
{ "application": { "type": "m.call" } }
```

### Slots (`m.rtc.slot`)

A **slot** is a virtual meeting room — a container for RTC members. Identified by a `slot_id` (state key of the `m.rtc.slot` event). Slots require elevated power levels to create/close; joining only requires normal user power.

Slot ID grammar: `{application.type}#{application_slot_id}` (e.g. `m.call#ROOM`).

States: Closed → Open → Active (≥1 member connected) / Inactive (0 members).

Slots can be:
- **Long-lived** (Discord-style, always-on spaces).
- **Scheduled** (managed via delayed events MSC4140 or a bot).
- **Ephemeral** (standard call flow).

### Membership (`m.rtc.member`)

Participant state is represented as a **sticky event** ([MSC4354](https://github.com/matrix-org/matrix-spec-proposals/pull/4354)). This gives room-state-like semantics without polluting room state permanently.

Key fields: `slot_id`, `application`, `member` (id, device_id, user_id), `rtc_transports`, `sticky_key`, `versions`.

A participant is **connected** if:
- The slot is open and the event's `sticky_key` matches `member.id`.
- The sticky event has not expired.
- The sender is still a room member.

### Sessions

A **MatrixRTC session** is the period of overlapping Connected memberships within a slot. Session start = first member connects; session end = last member disconnects.

Sessions are reconstructed retrospectively from the room DAG — no single session-start event is emitted.

### Transport Discovery

Homeservers expose available RTC transports at:

```
GET /_matrix/client/v1/rtc/transports
```

(Unstable: `/_matrix/client/unstable/org.matrix.msc4143/rtc/transports`)

Response lists transport objects. Each transport has a `type` and type-specific fields. The primary transport is LiveKit ([MSC4195](https://github.com/matrix-org/matrix-spec-proposals/pull/4195)).

### End-to-End Encryption

In encrypted rooms, each participant shares a per-sender key via an **encrypted to-device message** (`m.rtc.encryption_key`). Keys are rotated on participant join/leave. A shared-key mode is also proposed for large calls.

---

## Companion MSCs

| MSC | Role |
|-----|------|
| [MSC4354](https://github.com/matrix-org/matrix-spec-proposals/pull/4354) | Sticky Events — membership representation (**hard dependency**) |
| [MSC4140](https://github.com/matrix-org/matrix-spec-proposals/pull/4140) | Delayed Events — deadman-switch for membership (**hard dependency**) |
| [MSC4195](https://github.com/matrix-org/matrix-spec-proposals/pull/4195) | LiveKit transport (**blocked on MSC4143**) |
| [MSC4196](https://github.com/matrix-org/matrix-spec-proposals/pull/4196) | `m.call` voice/video application (**blocked on MSC4143**) |
| [MSC4075](https://github.com/matrix-org/matrix-spec-proposals/pull/4075) | Call ringing / `m.rtc.notification` (**blocked on MSC4143**) |
| [MSC3401](https://github.com/matrix-org/matrix-spec-proposals/pull/3401) | Legacy full-mesh transport (planned future transport under MSC4143) |

---

## Open Issues / To-Dos

- Discovery and negotiation of available application types (left to a future MSC).
- Early media support (see [MSC3635](https://github.com/matrix-org/matrix-spec-proposals/pull/3635)).
- Inter-transport interoperability (left to a future MSC).
- Message ordering consistency for reliable session history reconstruction.
- SCT checklist not yet started; FCP tickyboxes not yet created.

---

## Unstable Prefixes

| Stable | Unstable |
|--------|---------|
| `m.rtc.member` | `org.matrix.msc4143.rtc.member` |
| `m.rtc.slot` | `org.matrix.msc4143.rtc.slot` |
| `m.rtc.encryption_key` | `org.matrix.msc4143.rtc.encryption_key` |
| `/_matrix/client/v1/rtc/transports` | `/_matrix/client/unstable/org.matrix.msc4143/rtc/transports` |

---

## References

- PR: https://github.com/matrix-org/matrix-spec-proposals/pull/4143
- Rendered proposal: https://github.com/matrix-org/matrix-spec-proposals/blob/toger5/matrixRTC/proposals/4143-matrix-rtc.md
