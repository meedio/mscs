# MSC4075: MatrixRTC `m.rtc.notification` — Call Ringing and Notifications

## Summary

MSC4075 defines **`m.rtc.notification`** — the event type used for pre-call signalling in MatrixRTC. This covers the ring/decline/busy flow that users expect from a traditional calling experience: ringing an incoming call on a recipient's device, allowing them to decline, and notifying participants of call state changes before they have joined a MatrixRTC slot.

---

## Status

| Field | Value |
|-------|-------|
| **PR** | [#4075](https://github.com/matrix-org/matrix-spec-proposals/pull/4075) |
| **State** | Open |
| **Author** | (Element team) |
| **Labels** | `voip`, `matrix-2.0` |
| **FCP** | Not started |
| **Lifecycle stage** | Draft; pending MSC4143 and MSC4196 |

---

## Role in the MatrixRTC Stack

MSC4075 fills the pre-join signalling gap in the MatrixRTC model:

```
MSC4143 handles:   in-session membership and transport
MSC4196 handles:   in-session UX and stream semantics for m.call
MSC4075 handles:   pre-session signalling (ringing, decline, busy)
```

The `m.rtc.notification` event is sent as a to-device message or room message (TBD) before a participant has sent an `m.rtc.member` event. It allows:
- **Caller** to ring the recipient's devices.
- **Recipient** to decline before joining.
- **All parties** to be notified of call state (e.g. missed call).

---

## Dependency Chain

```
MSC4143 (MatrixRTC base)
  ├─► MSC4075 (notifications/ringing)  ← this MSC
  └─► MSC4196 (m.call application)
        └─► MSC4075 (also required by MSC4196)
```

---

## References

- PR: https://github.com/matrix-org/matrix-spec-proposals/pull/4075
