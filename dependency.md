# MSC Dependency Map

This document maps the interconnections between all MatrixRTC-related MSCs and the order in which they must be completed (merged/accepted) for the MatrixRTC stack to be fully specified.

---

## ⚡ Live Status (May 21, 2026)

- **MSC4140** is now **"Proposed for FCP readiness"** — active spec review by @Johennes; @turt2live says "looking closer to being ready for FCP".
- **MSC4354** is under **comprehensive review right now** — @turt2live approved; @richvdh has requested changes.
- **MSC4143** received a **9-task SCT checklist** from @turt2live on May 19, 2026.
- **MSC4480** (Sliding Sync extension for Sticky Events) was split out from MSC4354 on May 20, 2026 — new dependency.

## Dependency Graph

```
Layer 0 — Foundations (must land first)
┌──────────────────────────────────────────────────────────────┐
│  MSC4140: Cancellable Delayed Events                         │
│  (heartbeat / deadman-switch for membership)                 │
│  Status: ⬆️ "Proposed for FCP readiness" (May 2026)         │
└──────────────────────────────────────────────────────────────┘
         │
         │ (independently progressing in parallel)
         │
┌──────────────────────────────────────────────────────────────┐
│  MSC4354: Sticky Events                                      │
│  (ephemeral persistent-state primitive for membership)       │
│  Status: Active review; @turt2live approved; @richvdh reqs  │
│                                                              │
│  → MSC4480: Sliding Sync Extension (split out May 20, 2026)  │
└──────────────────────────────────────────────────────────────┘
         │
         │ (both must land before MSC4143 can proceed)
         ▼
Layer 1 — Core Framework
┌──────────────────────────────────────────────────────────────┐
│  MSC4143: MatrixRTC                                          │
│  (slots, membership, sessions, transport discovery, E2EE)    │
│  Status: Open — BLOCKED on MSC4354 + MSC4140                 │
│  SCT: 9-task checklist created May 19, 2026 by @turt2live    │
└──────────────────────────────────────────────────────────────┘
         │
         ├────────────────────┬────────────────────┐
         ▼                    ▼                    ▼
Layer 2 — Transport / Application / Notifications
┌─────────────────┐  ┌─────────────────┐  ┌──────────────────┐
│ MSC4195:        │  │ MSC4196:        │  │ MSC4075:         │
│ LiveKit         │  │ m.call app      │  │ RTC notifications│
│ Transport       │  │ (voice/video)   │  │ (ringing)        │
│                 │  │                 │  │  └── MSC4310:    │
│ Status: Open    │  │ Status: Open    │  │      m.rtc.      │
│ BLOCKED on      │  │ BLOCKED on      │  │      decline     │
│ MSC4143         │  │ MSC4143+MSC4075 │  │ BLOCKED on       │
└─────────────────┘  └─────────────────┘  │ MSC4143          │
                                           └──────────────────┘
```

### Maintenance MSCs (parallel track)

```
MSC4140 ──► MSC4309: Finalised Delayed Event on Sync
            (maintenance; adds sync notification when delayed event fires)
            Status: Open — early draft, needs-implementation

MSC4140 ──► MSC4479: Shorten alternatives  [MERGED May 20, 2026]
MSC4140 ──► MSC4478: Move use cases section out  [In progress May 2026]

MSC4354 ──► MSC4480: Sliding Sync Extension: Sticky Events  [Open May 20, 2026]
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
1. [IN PROGRESS] Resolve @richvdh's requested changes in MSC4354 → enter FCP → merge MSC4354
2. [IN PROGRESS] Complete spec language review of MSC4140 → enter FCP → merge MSC4140
   - MSC4479 already merged; MSC4478 in progress
   - Status: "Proposed for FCP readiness" as of May 2026
3. (Steps 1 and 2 can happen in parallel)
4. Once MSC4354 and MSC4140 are merged: MSC4143 implementation testing
   - SCT has already created the 9-task checklist (May 19, 2026)
5. MSC4143 SCT checklist + FCP → merge MSC4143
6. MSC4195 (transport), MSC4075 (notifications), MSC4196 (m.call) can now enter FCP
7. MSC4196 depends on both MSC4143 AND MSC4075 landing first
```

**Estimated timeline (based on current activity):** FCP for MSC4140 and MSC4354 in Q2–Q3 2026; MSC4143 FCP in Q3–Q4 2026 if the pace holds. Matrix 2.0 spec bump would follow shortly after.

---

## Per-MSC Dependency Table

| MSC | Depends On | Blocked By | Blocks | Status (May 2026) |
|-----|-----------|------------|--------|-------------------|
| MSC4354 Sticky Events | — | @richvdh's requested changes | MSC4143, MSC4480 | Active review |
| MSC4480 Sliding Sync for Sticky Events | MSC4354 | MSC4354 | — | Just opened (May 20) |
| MSC4140 Delayed Events | — | Spec review in progress | MSC4143, MSC4309 | **Proposed for FCP** |
| MSC4479 Shorten MSC4140 alternatives | MSC4140 | — | — | **Merged** (May 20) |
| MSC4478 MSC4140 use cases | MSC4140 | — | — | In progress (May 2026) |
| MSC4309 Delayed Event on Sync | MSC4140 | MSC4140 | — | Early draft |
| MSC4143 MatrixRTC | MSC4354, MSC4140 | MSC4354, MSC4140 | MSC4195, MSC4075, MSC4196 | BLOCKED; SCT checklist created |
| MSC4195 LiveKit Transport | MSC4143 | MSC4143 | — | Open |
| MSC4075 RTC Notifications | MSC4143 | MSC4143 | MSC4196, MSC4310 | Open; needs impl update |
| MSC4310 m.rtc.decline | MSC4075 | MSC4075 | — | Open; has implementations |
| MSC4196 m.call Application | MSC4143, MSC4075 | MSC4143, MSC4075 | — | Draft |
| MSC3401 Group VoIP | — | (largely superseded) | — | Historical |
| MSC3898 SFU Cascading | MSC3401 | (dormant) | — | Dormant |

---

## What Needs to Happen to Unblock Everything

### Unblock MSC4354 (Sticky Events)

- Resolve the open concerns tracked under `unresolved-concerns` label in [PR #4354](https://github.com/matrix-org/matrix-spec-proposals/pull/4354).
- Propose FCP to SCT in [#sct-office:matrix.org](https://matrix.to/#/#sct-office:matrix.org).

### Unblock MSC4140 (Delayed Events)

- Address remaining @Johennes spec review comments (wording, `running_since` → `scheduled_ts`, etc).
- MSC4478 (use cases section) needs to merge.
- `implementation-needs-checking` needs SCT sign-off.
- Propose FCP to SCT. **Status: "Proposed for FCP readiness" as of May 2026.**

### After MSC4354 + MSC4140 land

- MSC4143 implementation testing can proceed fully.
- SCT checklist for MSC4143 exists (9 tasks created May 19, 2026) and must be worked through.
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
