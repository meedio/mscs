# MSC4196: MatrixRTC Voice and Video Calling Application (`m.call`)

## Summary

MSC4196 defines the **`m.call` application type** — the first first-class MatrixRTC application, specifying how voice and video conferencing works on top of the MSC4143 framework. It defines the specific metadata, signalling, and UX for calling, including how clients interpret audio/video streams, how call state is managed, and how the calling experience integrates with familiar "join/leave call" UX patterns.

---

## Status

| Field | Value |
|-------|-------|
| **PR** | [#4196](https://github.com/matrix-org/matrix-spec-proposals/pull/4196) |
| **State** | Open (draft) |
| **Author** | (Element team) |
| **Created** | September 19, 2024 |
| **Last updated** | January 12, 2026 (major rewrite addressing offline feedback) |
| **Labels** | `matrix-2.0`, `voip`, `proposal` |
| **FCP** | Not started |
| **Lifecycle stage** | Draft / active development, pending MSC4143 |

---

## Scope

MSC4196 defines:

- The `m.call` application type and its JSON schema in `m.rtc.slot` and `m.rtc.member` events.
- How voice/video streams are labelled and interpreted (audio tracks, video tracks, screen-share tracks).
- How clients should present call UX on top of the MatrixRTC slot/membership model (e.g. "Call" button in room header, ongoing call indicator).
- Call-specific `m.rtc.slot` constraints such as `m.call.voice_only`.

MSC4196 does **not** define:
- Pre-call signalling (ringing, declining) — that is handled by [MSC4075](https://github.com/matrix-org/matrix-spec-proposals/pull/4075).
- The underlying transport — that is MSC4195 (LiveKit) or MSC3401 (full-mesh).
- The session-level RTC protocol — that is MSC4143.

---

## Dependencies

```
MSC4143 (MatrixRTC base)
  └─► MSC4196 (m.call application) — blocked on MSC4143
        └─► MSC4075 (call ringing/notifications) — also required
```

MSC4196 cannot be accepted until MSC4143 is accepted.

---

## Open To-Dos (as of Jan 2026)

- Clarify how to distinguish between voice-only and video calls in the slot/member event.
- Finalize field structures for the application metadata.

---

## Ecosystem Note

Element Call and matrix-js-sdk already implement a version of `m.call` behaviour using the unstable `org.matrix.msc3401.call` prefix. The Cinny client added video call support referencing MSC4196 as of February 2026.

---

## References

- PR: https://github.com/matrix-org/matrix-spec-proposals/pull/4196
