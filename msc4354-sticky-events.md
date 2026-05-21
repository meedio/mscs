# MSC4354: Sticky Events

## Summary

MSC4354 introduces **Sticky Events** — a new Matrix communication primitive that provides per-user last-write-wins ephemeral state with guaranteed delivery, federation push, and access control tied to room membership. It was designed specifically to replace the use of room state events for MatrixRTC membership (`m.rtc.member`), addressing persistent problems with state event abuse, rollbacks, and spam.

---

## Status

| Field | Value |
|-------|-------|
| **PR** | [#4354](https://github.com/matrix-org/matrix-spec-proposals/pull/4354) |
| **State** | Open |
| **Author** | [@kegsay](https://github.com/kegsay) (Kegan Dougal, Element) |
| **Created** | September 16, 2025 |
| **Last updated** | April 1, 2026 |
| **Labels** | `proposal`, `client-server`, `kind:core`, `unresolved-concerns`, `matrix-2.0` |
| **FCP** | FCP tickyboxes exist; `unresolved-concerns` label present — not yet in FCP countdown |
| **Lifecycle stage** | Implementations done; resolving outstanding concerns |

### Implementations

| Component | Status | Link |
|-----------|--------|------|
| Client (receive/handle) — matrix-js-sdk | ✅ Done | [matrix-org/matrix-js-sdk#5017](https://github.com/matrix-org/matrix-js-sdk/pull/5017) |
| Client (usage) — Element Call | ✅ Done | [element-hq/element-call#3513](https://github.com/element-hq/element-call/pull/3513) |
| Server — Synapse | ✅ Done | [element-hq/synapse#18968](https://github.com/element-hq/synapse/pull/18968) |
| Tests — Complement | ✅ Done | [matrix-org/complement#806](https://github.com/matrix-org/complement/pull/806) |

---

## Why Sticky Events Were Needed

Problems with using room state for per-user per-device temporary data (e.g. call membership):

1. **Any user can overwrite another user's state** — MSC3757 tried to fix this with string-packing auth, which is architecturally awkward.
2. **Unprivileged users can spam room state indefinitely** — the DAG is append-only; old state accumulates forever.
3. **State resolution causes rollbacks** — conflicting state events can silently roll back a participant's membership state.

EDUs (receipts/typing) are also inadequate because they are not extensible and do not guarantee delivery across federation.

---

## Proposal

Message events can carry a new top-level `sticky` object:

```json
{
  "type": "m.rtc.member",
  "sticky": { "duration_ms": 600000 },
  "sender": "@alice:example.com",
  "room_id": "!foo",
  "content": { ... }
}
```

- `duration_ms`: integer 0–3600000 (max 1 hour). The event is "sticky" for this many milliseconds after its start time.
- Start time = `min(received_ts, origin_server_ts)`.
- End time = `start_time + min(sticky_duration_ms, 3600000)`.

### Setting stickiness (Client-Server API)

A `sticky_duration_ms` query parameter is added to:
- `PUT /_matrix/client/v3/rooms/{roomId}/send/{eventType}/{txnId}`
- `PUT /_matrix/client/v3/rooms/{roomId}/state/{eventType}/{stateKey}`

### Guarantees while sticky

Servers MUST:
1. **Push** sticky events eagerly to all joined servers (with backoff; never drop).
2. **Push** all own sticky events to any newly joining server.
3. **Deliver** sticky events to clients via `/sync` in a new `sticky.events` section, regardless of timeline window.
4. Re-evaluate **soft-failure** when membership changes for a user with unexpired sticky events.
5. Skip **history visibility** checks — any joined member can see sticky events for their sticky duration.

### Sync API change

New section in the join room sync response:

```json
{
  "rooms": {
    "join": {
      "!room:example.com": {
        "sticky": {
          "events": [
            {
              "sender": "@bob:example.com",
              "type": "m.rtc.member",
              "sticky": { "duration_ms": 600000 },
              "unsigned": { "sticky_duration_ttl_ms": 258113 },
              "content": { ... }
            }
          ]
        }
      }
    }
  }
}
```

### Key-value store semantics (for MatrixRTC)

An addendum defines a key-value store mode: the `sticky_key` field in the event content makes the server track the latest event per `(sender, type, sticky_key)` tuple, giving the same convergence semantics as Matrix room state without the state bloat.

---

## Relationship to Other MSCs

| MSC | Relationship |
|-----|-------------|
| [MSC4143](https://github.com/matrix-org/matrix-spec-proposals/pull/4143) | Uses sticky events for `m.rtc.member` — **hard dependency** |
| [MSC4140](https://github.com/matrix-org/matrix-spec-proposals/pull/4140) | Can be combined with sticky events for heartbeat semantics |
| [MSC3489](https://github.com/matrix-org/matrix-spec-proposals/pull/3489) | Live location sharing — same problem, could use sticky events |
| [MSC3757](https://github.com/matrix-org/matrix-spec-proposals/pull/3757) | Earlier attempt to restrict state event overwriting — sticky events supersede this need for RTC |
| [MSC2477](https://github.com/matrix-org/matrix-spec-proposals/pull/2477) | Custom EDU types — alternative approach rejected in favour of sticky events |

---

## Outstanding Concerns

The `unresolved-concerns` label indicates there are open issues that must be resolved before FCP can proceed. Specific concerns as of April 2026 are tracked in the PR thread.

---

## References

- PR: https://github.com/matrix-org/matrix-spec-proposals/pull/4354
- Rendered proposal: https://github.com/matrix-org/matrix-spec-proposals/blob/kegan/persist-edu/proposals/4354-sticky-events.md
