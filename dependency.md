# MSC Dependency Map

This document maps the interconnections between all MatrixRTC-related MSCs and the order in which they must be completed (merged/accepted) for the MatrixRTC stack to be fully specified.

---

## Dependency Graph

```
Layer 0 — Foundations (must land first)
┌──────────────────────────────────────────────────────────────┐
│  MSC4140: Cancellable Delayed Events                         │
│  (heartbeat / deadman-switch for membership)                 │
│  Status: Open — FCP not started, known impl gaps             │
└──────────────────────────────────────────────────────────────┘
         │
         │ (independently progressing in parallel)
         │
┌──────────────────────────────────────────────────────────────┐
│  MSC4354: Sticky Events                                      │
│  (ephemeral persistent-state primitive for membership)       │
│  Status: Open — unresolved-concerns, FCP tickyboxes exist    │
└──────────────────────────────────────────────────────────────┘
         │
         │ (both must land before MSC4143 can proceed)
         ▼
Layer 1 — Core Framework
┌──────────────────────────────────────────────────────────────┐
│  MSC4143: MatrixRTC                                          │
│  (slots, membership, sessions, transport discovery, E2EE)    │
│  Status: Open — BLOCKED on MSC4354 + MSC4140                 │
└──────────────────────────────────────────────────────────────┘
         │
         ├────────────────────┬────────────────────┐
         ▼                    ▼                    ▼
Layer 2 — Transport / Application / Notifications
┌─────────────────┐  ┌─────────────────┐  ┌──────────────────┐
│ MSC4195:        │  │ MSC4196:        │  │ MSC4075:         │
│ LiveKit         │  │ m.call app      │  │ RTC notifications│
│ Transport       │  │ (voice/video)   │  │ (ringing)        │
│                 │  │                 │  │                  │
│ Status: Open    │  │ Status: Open    │  │ Status: Open     │
│ BLOCKED on      │  │ BLOCKED on      │  │ BLOCKED on       │
│ MSC4143         │  │ MSC4143+MSC4075 │  │ MSC4143          │
└─────────────────┘  └─────────────────┘  └──────────────────┘
```

### Maintenance MSC (parallel track)

```
MSC4140 ──► MSC4309: Finalised Delayed Event on Sync
            (maintenance; adds sync notification when delayed event fires)
            Status: Open — early draft, needs-implementation
```

### Legacy / Superseded (informational)

```
MSC3401: Group VoIP (full-mesh)
  → Partially superseded by MSC4143; planned as a future full-mesh transport
  → No blocking relationship to current MatrixRTC stack

MSC3898: Cascading SFU Signalling
  → Built on top of MSC3401; largely dormant
  → Superseded in practice by MSC4195's federated multi-SFU approach
```

---

## Critical Path to MatrixRTC Acceptance

For the core MatrixRTC spec to be complete and merged, the following must happen **in order**:

```
1. Resolve unresolved-concerns in MSC4354 → enter FCP → merge MSC4354
2. Close known implementation gaps in MSC4140 → enter FCP → merge MSC4140
3. (Steps 1 and 2 can happen in parallel)
4. Once MSC4354 and MSC4140 are merged: MSC4143 implementation testing
5. MSC4143 SCT checklist + FCP → merge MSC4143
6. MSC4195 (transport), MSC4075 (notifications), MSC4196 (m.call) can now enter FCP
7. MSC4196 depends on both MSC4143 AND MSC4075 landing first
```

---

## Per-MSC Dependency Table

| MSC | Depends On | Blocked By | Blocks |
|-----|-----------|------------|--------|
| MSC4354 Sticky Events | — | unresolved-concerns | MSC4143 |
| MSC4140 Delayed Events | — | impl gaps | MSC4143, MSC4309 |
| MSC4309 Delayed Event on Sync | MSC4140 | MSC4140 | — |
| MSC4143 MatrixRTC | MSC4354, MSC4140 | MSC4354, MSC4140 | MSC4195, MSC4075, MSC4196 |
| MSC4195 LiveKit Transport | MSC4143 | MSC4143 | — |
| MSC4075 RTC Notifications | MSC4143 | MSC4143 | MSC4196 |
| MSC4196 m.call Application | MSC4143, MSC4075 | MSC4143, MSC4075 | — |
| MSC3401 Group VoIP | — | (largely superseded) | — |
| MSC3898 SFU Cascading | MSC3401 | (dormant) | — |

---

## What Needs to Happen to Unblock Everything

### Unblock MSC4354 (Sticky Events)

- Resolve the open concerns tracked under `unresolved-concerns` label in [PR #4354](https://github.com/matrix-org/matrix-spec-proposals/pull/4354).
- Propose FCP to SCT in [#sct-office:matrix.org](https://matrix.to/#/#sct-office:matrix.org).

### Unblock MSC4140 (Delayed Events)

- Close the three known implementation gaps (commits `3ef314f`, `95045cf`, `49b200d`).
- Get `implementation-needs-checking` resolved.
- Propose FCP to SCT.

### After MSC4354 + MSC4140 land

- MSC4143 implementation testing can proceed fully.
- SCT checklist for MSC4143 must be filled in (currently empty).
- FCP for MSC4143 can be proposed.

### After MSC4143 lands

- MSC4195, MSC4075 unblock immediately (implementations already exist).
- MSC4196 unblocks after MSC4075 also lands.

---

## Conceptual Layering

```
┌──────────────────────────────────────────────────────┐
│          MatrixRTC Applications                      │
│  MSC4196 (m.call)    future apps (games, docs, etc.) │
├──────────────────────────────────────────────────────┤
│          Pre-call Signalling                         │
│  MSC4075 (m.rtc.notification — ringing)              │
├──────────────────────────────────────────────────────┤
│          RTC Transports                              │
│  MSC4195 (LiveKit)   MSC3401 (full-mesh, future)     │
├──────────────────────────────────────────────────────┤
│          Core Framework                              │
│  MSC4143 (slots, membership, sessions, E2EE, etc.)   │
├──────────────────────────────────────────────────────┤
│          Matrix Protocol Primitives                  │
│  MSC4354 (Sticky Events)  MSC4140 (Delayed Events)   │
└──────────────────────────────────────────────────────┘
```
