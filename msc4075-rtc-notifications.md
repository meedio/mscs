# MSC4075: MatrixRTC `m.rtc.notification` — Call Ringing and Notifications

## Summary

MSC4075 defines **`m.rtc.notification`** — the event type used for pre-call signalling in MatrixRTC. This covers the ring/decline/busy flow that users expect from a traditional calling experience: ringing an incoming call on a recipient's device, allowing them to decline, and notifying participants of call state changes before they have joined a MatrixRTC slot.

---

## Status

| Field | Value |
|-------|-------|
| **PR** | [#4075](https://github.com/matrix-org/matrix-spec-proposals/pull/4075) |
| **State** | Open |
| **Author** | [@toger5](https://github.com/toger5) (Timo K, Element) |
| **Created** | November 8, 2023 |
| **Last major update** | July 2025 (major rework: renamed event, added `lifetime`, added `m.reference` relation) |
| **Commits** | 19 |
| **Additions** | +227 lines |
| **Branch** | `toger5/matrixrtc-call-ringing` |
| **Labels** | `kind:feature`, `matrix-2.0`, `needs-implementation`, `proposal`, `voip` |
| **FCP** | Not started |
| **SCT Project Status** | "Tracking for review" |
| **Lifecycle stage** | Active draft; revised format needs implementation catch-up |

---

### Implementations

| Component | Status | Link |
|-----------|--------|------|
| Element Web (sending rings) | ✅ Done | [matrix-react-sdk#11870](https://github.com/matrix-org/matrix-react-sdk/pull/11870) |
| Ruma (original format) | ✅ Done | [ruma/ruma#1704](https://github.com/ruma/ruma/pull/1704) |
| Ruma (updated July 2025 format) | ✅ Done | [ruma/ruma#2199](https://github.com/ruma/ruma/pull/2199) |
| matrix-rust-sdk (audio/video intent) | ✅ Done | [matrix-rust-sdk#6207](https://github.com/matrix-org/matrix-rust-sdk/pull/6207), [#6412](https://github.com/matrix-org/matrix-rust-sdk/pull/6412) |
| Element X Android (new format) | ✅ Done | [element-x-android#5357](https://github.com/element-hq/element-x-android/pull/5357) |
| matrix-js-sdk (revised format with relations) | 🔄 In progress | [matrix-js-sdk#4826](https://github.com/matrix-org/matrix-js-sdk/pull/4826) |
| Element X iOS (ringing/rings) | 🔄 Partial | Tracked in [element-x-ios#5097](https://github.com/element-hq/element-x-ios/issues/5097) |
| Extensible events fallback (EW, EX iOS, EX Android) | ⏳ Pending | Not yet implemented |

---

## Proposal

### Event structure (as of July 2025 rework)

```json
{
  "type": "m.rtc.notification",
  "content": {
    "sender_ts": 1752583130365,
    "lifetime": 30000,
    "m.mentions": { "user_ids": [], "room": true },
    "m.relates_to": {
      "rel_type": "m.reference",
      "event_id": "$rtc_member_event_id"
    },
    "notification_type": "ring | notification",
    "m.call.intent": "voice | video"
  }
}
```

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `sender_ts` | integer | Client-local timestamp when the notification was sent |
| `lifetime` | integer (ms) | How long the receiver should ring before auto-stopping |
| `m.mentions` | object | Standard Matrix mentions — controls push rules and who is notified |
| `m.relates_to` | object | `m.reference` relation pointing to the sender's `m.rtc.member` event |
| `notification_type` | string | `"ring"` = ring the device; `"notification"` = silent push notification |
| `m.call.intent` | string | **Optional** — `"voice"` or `"video"`. Already in production implementations; needs formalising in proposal text |

### Client behaviour on receiving

A client SHOULD present a ringing UI if ALL of the following apply:
1. The `m.rtc.notification` event is in the room.
2. The notification is recent (within `lifetime` ms).
3. The local user is mentioned (via `m.mentions`).
4. The referenced `m.rtc.member` event is still active (the caller has not left).

Clients SHOULD NOT render the `m.rtc.notification` event in the timeline. Historic calls should be reconstructed from `m.rtc.member` session overlaps (time-based join/leave interval analysis).

---

## How Call Decline Works

Call decline is handled by a companion MSC:

**[MSC4310](https://github.com/matrix-org/matrix-spec-proposals/pull/4310): MatrixRTC decline `m.rtc.decline`**
- Defines an `m.rtc.decline` event that references the notification event via `m.reference`
- Created July 2025
- Implementations: matrix-rust-sdk ✅, Ruma ✅, matrix-js-sdk 🔄
- Status: Open draft; `matrix-2.0` label

---

## Open Issues in the Proposal

1. **`m.call.intent` field** — in all production implementations (js-sdk, rust-sdk, Ruma) but not yet in the formal proposal text. Naming may change to `m.rtc.intent` (per @BillCarsonFr — since this MSC is about general RTC sessions, not just calls). Values: `"voice"` | `"video"`.
2. **`m.mentions` optionality** — @Susurrus raised: should `m.mentions` be required (with at least `room: true` or a user listed)? The proposal currently allows empty mentions.
3. **Timeline rendering** — @toger5's position: do NOT use this event for timeline rendering (use `m.rtc.member` history reconstruction). @Susurrus and @Fractal are implementing timeline display based on the notification event as a stopgap.
4. **Implementation catch-up** — The July 2025 rework changed the event structure significantly (`m.reference` relation, `lifetime`, rename). Some implementations still need updating to the new format.

---

## Role in the MatrixRTC Stack

```
User A wants to call User B
   │
   ├─► User A joins MatrixRTC slot (m.rtc.member via MSC4143)
   │
   └─► User A sends m.rtc.notification (this MSC)
         ├── lifetime: 30000  (ring for 30s)
         ├── m.mentions: { room: false, user_ids: ["@userB:example.com"] }
         ├── m.relates_to → User A's m.rtc.member event
         └── notification_type: "ring"

User B's device rings
   │
   ├── If accepted → User B joins the slot (m.rtc.member)
   └── If declined → User B sends m.rtc.decline (MSC4310)
```

---

## Dependency Chain

```
MSC4143 (MatrixRTC framework)
    └── MSC4075 (m.rtc.notification — ringing)
            └── MSC4310 (m.rtc.decline — declining)
    └── MSC4196 (m.call application — extends MSC4075 with m.call.intent)
```

---

## References

- PR: https://github.com/matrix-org/matrix-spec-proposals/pull/4075
- Rendered proposal: https://github.com/matrix-org/matrix-spec-proposals/blob/toger5/matrixrtc-call-ringing/proposals/4075-rtc-notification-event.md
- MSC4310 (call decline): https://github.com/matrix-org/matrix-spec-proposals/pull/4310
