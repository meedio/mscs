# PR #4143: MSC4143 MatrixRTC

**Source:** https://github.com/matrix-org/matrix-spec-proposals/pull/4143

## Metadata

- **State:** Open
- **Author:** @toger5 (Timo K, Element)
- **Created:** May 10, 2024
- **Last updated:** March 17, 2026
- **Branch:** `toger5/matrixRTC` → `main`
- **Commits:** 31
- **Additions:** +1,172 lines
- **Labels:** proposal, voip, client-server, matrix-2.0
- **Rendered proposal:** https://github.com/matrix-org/matrix-spec-proposals/blob/toger5/matrixRTC/proposals/4143-matrix-rtc.md

## PR Description

> Rendered
>
> To-do:
> - discovery and negotiation of available applications
> - how do you support early media? see MSC3635: Early Media for VoIP #3635
>
> SCT Stuff:
> No MSC checklist
> No FCP tickyboxes
>
> BLOCKED on:
> - Dependencies landing
>   - MSC4354: Sticky Events #4354
>   - MSC4140: Cancellable delayed events #4140
> - Implementation testing (after dependencies land/are stable enough).

## Key Timeline Events

- May 10, 2024: First commit, PR opened
- Sep–Oct 2024: Multiple major revisions incorporating slots concept, sticky events integration
- Sep 2025: MSC4354 opens, MSC4143 updated to use it
- Late 2025 – Mar 2026: Continued refinement of proposal text (E2EE, session history, slot management UX)
- March 2026: Latest update; still blocked on dependencies

## Notable Discussion Points

- Slots concept (virtual meeting rooms within a Matrix room) was a key architectural decision
- Sticky events chosen over room state for membership to avoid state bloat and rollbacks
- E2EE key rotation grace period / delay-before-use parameters discussed
- Session history reconstruction algorithm (using `origin_server_ts` as workaround for lack of `received_server_ts`)
- Shared key mode for large calls proposed as optional alternative to per-sender keys
