# PR #4195: MSC4195 MatrixRTC Transport using LiveKit Backend

**Source:** https://github.com/matrix-org/matrix-spec-proposals/pull/4195

## Metadata

- **State:** Open
- **Author:** @hughns (Hugh Nimmo-Smith, Element)
- **Created:** September 16, 2024
- **Last updated:** November 2025
- **Branch:** `hughns/matrixrtc-livekit` (fork: `hughns/matrix-spec-proposals`) → `matrix-org:main`
- **Commits:** 26
- **Additions:** +568 lines
- **Labels:** matrix-2.0
- **Rendered (author fork):** https://github.com/hughns/matrix-spec-proposals/blob/hughns/matrixrtc-livekit/proposals/4195-matrixrtc-livekit.md

## PR Description

```
Rendered

Implementation:
- Backend: MatrixRTC Authorisation Service
  - https://github.com/element-hq/lk-jwt-service/releases/tag/v0.4.1
- Client: Element Call
  - https://github.com/element-hq/element-call/releases/tag/v0.17.0-rc.1

Dependencies:
- MSC4143: MatrixRTC #4143

To-do:
- Define error responses for POST /get_token

SCT Stuff:
No MSC checklist
No FCP tickyboxes
BLOCKED on MSC4143
```

## Key Implementation Links

- lk-jwt-service v0.4.1 release: https://github.com/element-hq/lk-jwt-service/releases/tag/v0.4.1
  - Implements `POST /get_token` endpoint per MSC4195
  - Maintains legacy `/get/sfu` for backward compat
  - Adds unified error handling, `SFURequest` type
- Element Call v0.17.0-rc.1: https://github.com/element-hq/element-call/releases/tag/v0.17.0-rc.1
- matrix-js-sdk `_unstable_getRTCTransports`: https://github.com/matrix-org/matrix-js-sdk/commit/2218ec4e3102e841ba3e794e1c492c0a5aa6c1c3

## Technical Notes

- LiveKit room alias = `base64url(SHA-256(room_id + "#" + slot_id))`
- Homeserver advertises transport at `GET /_matrix/client/v1/rtc/transports` with `type: "livekit"`
- Each participant publishes to their own homeserver's SFU — federated multi-SFU by default
- No cascading required in the base case (unlike MSC3898)
