# MSC4195: MatrixRTC Transport using LiveKit Backend

## Summary

MSC4195 specifies the **LiveKit-based RTC transport** for MatrixRTC. It defines how homeservers advertise LiveKit infrastructure, how clients obtain JWT tokens to join LiveKit rooms, and how participants publish and subscribe to media streams via LiveKit SFUs. This is the primary (and currently only production-ready) transport for MSC4143-based MatrixRTC.

---

## Status

| Field | Value |
|-------|-------|
| **PR** | [#4195](https://github.com/matrix-org/matrix-spec-proposals/pull/4195) |
| **State** | Open |
| **Author** | [@hughns](https://github.com/hughns) (Hugh Nimmo-Smith, Element) |
| **Created** | September 16, 2024 |
| **Last updated** | November 2025 (latest PR activity) |
| **Labels** | `matrix-2.0` (and related) |
| **FCP** | Not started; no MSC checklist or FCP tickyboxes |
| **Lifecycle stage** | Implementation done; **blocked on MSC4143** |

### Implementations

| Component | Status | Link |
|-----------|--------|------|
| Backend — lk-jwt-service v0.4.1 | ✅ Done | [element-hq/lk-jwt-service v0.4.1](https://github.com/element-hq/lk-jwt-service/releases/tag/v0.4.1) |
| Client — Element Call v0.17.0-rc.1 | ✅ Done | [element-hq/element-call v0.17.0-rc.1](https://github.com/element-hq/element-call/releases/tag/v0.17.0-rc.1) |
| matrix-js-sdk | ✅ Done | `_unstable_getRTCTransports` method added [2218ec4](https://github.com/matrix-org/matrix-js-sdk/commit/2218ec4e3102e841ba3e794e1c492c0a5aa6c1c3) |

---

## Proposal

### Transport advertisement

Homeservers advertise the LiveKit transport at the `GET /_matrix/client/v1/rtc/transports` endpoint (defined in MSC4143):

```json
{
  "rtc_transports": [
    {
      "type": "livekit",
      "livekit_service_url": "https://matrix-rtc.example.com/livekit/jwt"
    }
  ]
}
```

- `type`: Must be `"livekit"`.
- `livekit_service_url`: URL of the MatrixRTC Authorization Service (lk-jwt-service), which issues JWT tokens for LiveKit rooms.

### Token acquisition

The client posts to the `livekit_service_url` to obtain a JWT:

```
POST {livekit_service_url}/get_token
```

Request body (new MSC4195 format):
```json
{
  "matrix_room_id": "!room:example.com",
  "device_id": "ABCDEF",
  "member_id": "xyzABCDEF0123"
}
```

The service validates the Matrix access token, checks room membership, then returns a LiveKit JWT that grants publish/subscribe access to the appropriate LiveKit room.

### LiveKit room alias

The LiveKit room name is derived as:

```
base64url(SHA-256(matrix_room_id || "#" || slot_id))
```

Optional extra random bits can be appended for enhanced pseudonymity.

### Multi-SFU approach

Each participant publishes directly to **their own LiveKit SFU** (the one their homeserver provides) and subscribes to streams from other participants' SFUs. This is a **federated multi-SFU model** — there is no SFU election or cascading required in the basic case. Participants advertise their SFU in `rtc_transports` within their `m.rtc.member` event.

### Backward compatibility

The lk-jwt-service maintains a legacy `/get/sfu` endpoint alongside the new `/get_token` endpoint to support older clients during the transition.

---

## Open To-Dos

- Define error responses for `POST /get_token`.
- Full spec language for client behaviour when the token endpoint is unavailable.

---

## Dependency Chain

```
MSC4143 (MatrixRTC base)
  └─► MSC4195 (LiveKit transport) — blocked on MSC4143
```

MSC4195 cannot be accepted until MSC4143 is accepted.

---

## References

- PR: https://github.com/matrix-org/matrix-spec-proposals/pull/4195
- Rendered proposal (author's fork): https://github.com/hughns/matrix-spec-proposals/blob/hughns/matrixrtc-livekit/proposals/4195-matrixrtc-livekit.md
- lk-jwt-service (backend): https://github.com/element-hq/lk-jwt-service
- Element Call: https://github.com/element-hq/element-call
