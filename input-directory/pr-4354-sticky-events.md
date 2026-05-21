# PR #4354: MSC4354 Sticky Events

**Source:** https://github.com/matrix-org/matrix-spec-proposals/pull/4354

## Metadata

- **State:** Open
- **Author:** @kegsay (Kegan Dougal, Element)
- **Created:** September 16, 2025
- **Last updated:** April 1, 2026
- **Branch:** `kegan/persist-edu` → `main`
- **Commits:** 49
- **Additions:** +600 lines
- **Labels:** proposal, client-server, kind:core, unresolved-concerns, matrix-2.0
- **Rendered:** https://github.com/matrix-org/matrix-spec-proposals/blob/kegan/persist-edu/proposals/4354-sticky-events.md

## PR Description (Implementations section)

```
- [x] Client (receive/handle) https://github.com/matrix-org/matrix-js-sdk/pull/5017
- [x] Client (usage) https://github.com/element-hq/element-call/pull/3513
- [x] Server https://github.com/element-hq/synapse/pull/18968
- [x] Complement Tests https://github.com/matrix-org/complement/pull/806

SCT Stuff:
FCP tickyboxes: https://github.com/matrix-org/matrix-spec-proposals/pull/4354#issuecomment-3353839132
MSC checklist: https://github.com/matrix-org/matrix-spec-proposals/pull/4354#issuecomment-3353835809
```

## Key Reviewers

- @ara4n (Matthew Hodgson) — reviewed Sep 19, 2025
- @turt2live — added labels Sep 16, 2025
- @Johennes — reviewed Sep 17, 2025
- @BillCarsonFr — reviewed Sep 24, 2025
- @erikjohnston — reviewed Sep 24, 2025
- @toger5 — reviewed Sep 23, 2025 (as primary consumer for MatrixRTC)

## Notable Context

- Specifically designed to replace `m.call.member` state events from MSC3401
- Adds `sticky.events` section to `/sync` response
- Key-value store semantics via `sticky_key` field enables room-state-like convergence
- Max 1 hour stickiness (`duration_ms` 0–3600000)
- Server eagerly pushes to all joined servers (unlike EDUs which have no guarantee)
- History visibility does not apply — all joined members see sticky events for their duration
