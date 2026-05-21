# MSC4309: Finalised Delayed Event on Sync

## Summary

MSC4309 is a **maintenance MSC** building on MSC4140. It specifies how a client learns — via the `/sync` response — that a delayed event it scheduled has been **fired** (sent by the homeserver). Without this, clients have no in-band mechanism to know a delayed event fired while they were disconnected.

---

## Status

| Field | Value |
|-------|-------|
| **PR** | [#4309](https://github.com/matrix-org/matrix-spec-proposals/pull/4309) |
| **State** | Open |
| **Author** | (Element team) |
| **Created** | July 10, 2025 |
| **Last updated** | August 20, 2025 |
| **Labels** | `kind:maintenance`, `needs-implementation` |
| **FCP** | Not started |
| **Lifecycle stage** | Early draft |

---

## Problem

When a client schedules a delayed event (MSC4140), it periodically restarts the timer. If the client disconnects and reconnects, the delayed event may have already fired. The client currently has no way to know this from `/sync` — it must poll `GET /_matrix/client/v3/delayed_events` to check.

MSC4309 adds a compact notification in the `/sync` response when a delayed event fires, so clients can react appropriately (e.g. by knowing they were considered disconnected from a MatrixRTC session and need to rejoin).

---

## Dependency

- **Depends on:** [MSC4140](https://github.com/matrix-org/matrix-spec-proposals/pull/4140) (Cancellable Delayed Events)

---

## References

- PR: https://github.com/matrix-org/matrix-spec-proposals/pull/4309
