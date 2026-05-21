# MSC3401: Native Group VoIP Signalling

## Summary

MSC3401 was the original proposal for **multi-participant (group) VoIP signalling** in Matrix, using a full-mesh WebRTC topology. It introduced the concept of representing call membership as room state events (`m.call.member`) with per-device data. While still formally open, it has been substantially superseded by MSC4143 (MatrixRTC), which addresses its architectural limitations. It is now planned as a **future full-mesh transport** within the MSC4143 framework, not as a standalone replacement.

---

## Status

| Field | Value |
|-------|-------|
| **PR** | [#3401](https://github.com/matrix-org/matrix-spec-proposals/pull/3401) |
| **State** | Open (largely superseded) |
| **Author** | [@ara4n](https://github.com/ara4n) (Matthew Hodgson, Element/Matrix.org) |
| **Created** | September 19, 2021 |
| **Labels** | (voip, proposal) |
| **FCP** | Not started |
| **Lifecycle stage** | Active but superseded in scope by MSC4143 |

---

## Original Proposal

MSC3401 introduced:

- **`m.call.member` state events** — one per user, containing an array of per-device call memberships. Users join a call by sending this state event; they leave by clearing it.
- **`m.call` room state event** — defines a call taking place in the room.
- **Full-mesh WebRTC** — all participants establish direct peer-to-peer connections to each other.
- **SFU signalling** was split into the companion [MSC3898](https://github.com/matrix-org/matrix-spec-proposals/pull/3898).

This was the basis for the early Element Call implementation and shipped in limited form.

---

## Why It Was Superseded

MSC3401's approach has several architectural problems that MSC4143 addresses:

1. **Array-based membership** — one state event per user with an array of devices creates race conditions when multiple devices of the same user update the state concurrently. One device can overwrite another's membership.
2. **Delayed events incompatibility** — MSC4140 (delayed events) cannot reliably maintain an accurate membership state using arrays, because the current array content may change between when the delayed event is scheduled and when it fires.
3. **State bloat** — room state events accumulate indefinitely in the room DAG, creating long-term bloat.
4. **State resolution rollbacks** — concurrent state changes during a call can cause membership rollbacks that are catastrophic for real-time communication.

MSC4143 solves all of these by using **sticky events** (MSC4354) instead of room state for membership.

---

## Current Role in the MSC4143 Ecosystem

MSC3401 is referenced in MSC4143 as a **planned transport option**. The full-mesh WebRTC approach it describes will eventually be packaged as a MatrixRTC transport MSC (analogous to MSC4195 for LiveKit), allowing clients that prefer peer-to-peer to participate in MatrixRTC sessions without an SFU.

As of 2026, this full-mesh transport MSC has not been written.

---

## Implementations

MSC3401 was partially implemented in:
- Early versions of Element Call
- matrix-js-sdk (legacy call module)

These implementations are now being migrated to the MSC4143/MSC4195 architecture.

---

## Obsoletes

Replaces [MSC2359](https://github.com/matrix-org/matrix-spec-proposals/pull/2359) (earlier group VoIP proposal).

---

## References

- PR: https://github.com/matrix-org/matrix-spec-proposals/pull/3401
- Rendered proposal: https://github.com/matrix-org/matrix-spec-proposals/blob/matthew/group-voip/proposals/3401-group-voip.md
