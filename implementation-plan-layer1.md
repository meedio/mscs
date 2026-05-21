# Implementation Plan: MatrixRTC Layer 0 → Layer 1

**Goal:** MSC4354 merged + MSC4140 merged → MSC4143 merged (Layer 1 complete)  
**As of:** May 21, 2026  
**Scope:** Everything required for MSC4143 to pass FCP and merge into the Matrix spec.

---

## Executive Summary

Three parallel tracks must converge. Track A (MSC4140) is the furthest along — "Proposed for FCP readiness" as of this week. Track B (MSC4354) is close but has one hard unknown blocker (@richvdh's requested changes). Both tracks have a critical implementation gap in matrix-rust-sdk (sticky events). Track C (MSC4143) cannot start until A and B are complete, but the SCT has already prepared by creating a 9-task checklist.

The rust-sdk sticky events gap is the largest single risk: it blocks MSC4354's `implementation-needs-checking` label, and without it Element X cannot adopt the new membership format.

**Optimistic timeline:** MSC4140 FCP Q2 2026 → MSC4354 FCP Q3 2026 → MSC4143 FCP Q4 2026.  
**Risk scenario:** @richvdh's concerns on MSC4354 require significant rework → slip to 2027 H1.

---

## Track A — MSC4140: Cancellable Delayed Events

**Current status:** "Proposed for FCP readiness" in SCT project. @turt2live has requested changes. @Johennes is doing a comprehensive review pass (May 18–21, 2026). MSC4479 already merged. MSC4478 in progress.

---

### A-01 · Merge MSC4478: Move use cases section out of MSC4140

**Status:** In progress (opened by @Johennes, May 20, 2026)  
**Owner:** @Johennes / @toger5  
**Depends on:** Nothing  

The current proposal has a large "Use case specific considerations" section that reads almost like a separate proposal. @turt2live explicitly asked for it to be removed/shortened because it extends review surface significantly. MSC4478 extracts this into its own companion PR.

**What to do:** Review and merge MSC4478. Ensure the core MSC4140 text still references use cases briefly (e.g. "MatrixRTC uses this for deadman-switch; future MSCs may use it for self-destructing messages") without detailing them.

**Tradeoff:** Removing the use cases section makes MSC4140 harder to understand in isolation, but reduces review load — the right call for FCP progress.

---

### A-02 · Resolve `running_since` → `scheduled_ts` field rename

**Status:** Open discussion, agreement reached in principle  
**Owner:** @toger5 / @AndrewFerr  
**Depends on:** Nothing  

The `running_since` field in the `GET /delayed_events` response is ambiguous — it reads like a duration rather than a timestamp. @Johennes proposed renaming to `scheduled_ts` (May 18, 2026). @AndrewFerr agreed (`scheduled_ts` is best as it reuses terminology from the proposal).

**What to do:**
1. Rename `running_since` → `scheduled_ts` in the proposal text.
2. Update all implementations: Synapse, matrix-js-sdk, matrix-rust-sdk (widget driver), matrix-widget-api.
3. The implementations can be updated in parallel — no coordination required since this is still unstable-prefixed.

**Tradeoff:** Breaking change to the unstable API. Clients in production using `running_since` will need updating before MSC4140 stabilises. Given the small number of implementations, this is low-risk.

**Workaround if skipped:** Could define `scheduled_ts` as an alias and deprecate `running_since` after stabilisation, but this adds spec debt — better to fix now.

---

### A-03 · Resolve `M_MAX_DELAY_EXCEEDED` vs `/capabilities` decision

**Status:** Open — active discussion May 2026  
**Owner:** @AndrewFerr / @Johennes  
**Depends on:** Nothing  

Currently the proposal defines a custom `M_MAX_DELAY_EXCEEDED` error code for when a requested delay exceeds the server's limit. The alternative: expose `max_delay` via `/capabilities`, making the error unnecessary (client would check capabilities first and never send an over-limit request).

@AndrewFerr (May 20, 2026): "Yes, the point of a new error code was to be able to have the `max_delay` field in the error object. But this can now all be removed in favour of expressing the max delay in `/capabilities`."

**What to do:**
1. Add `m.delayed_events.max_delay_ms` (or similar) to `/capabilities` response.
2. Remove `M_MAX_DELAY_EXCEEDED` from the proposal.
3. Update Synapse to expose max delay in capabilities.
4. Update client implementations to check capabilities before scheduling.

**Tradeoff:** More roundtrips for first-time use (client must fetch capabilities before knowing if its delay is valid). Mitigated by capabilities caching. This is clearly the cleaner design.

**Risk:** Capabilities is a server-level capability. If a homeserver operator configures a very low max delay, clients that don't check capabilities will just get `M_INVALID_PARAM`. The error message should be human-readable enough to diagnose.

---

### A-04 · Address @turt2live's requested changes + @Johennes' review comments

**Status:** In progress — @turt2live requested changes May 19, 2026  
**Owner:** @toger5 / @AndrewFerr  
**Depends on:** A-01, A-02, A-03 (should be addressed together)  

Beyond the specific issues above, @turt2live has an overall "requested changes" blocking approval. The main asks:
- Remove/simplify the use cases section (→ A-01)
- Shorten the alternatives section (→ already done by MSC4479)
- @Johennes' wording review comments (delay_id security language, OAuth alternatives text, etc.)

**What to do:** Work through all remaining open @Johennes comments in the PR thread. Most are small wording suggestions with agreed changes already discussed. Estimate: 10–20 small commits.

**Note:** Many of these are co-authored commits between @toger5 and @Johennes (already 8 such commits merged May 19, 2026). The pace is fast.

---

### A-05 · SCT `implementation-needs-checking` sign-off for MSC4140

**Status:** Pending — label still present  
**Owner:** SCT (@turt2live, @Half-Shot, @hughns — awaiting review)  
**Depends on:** A-01 through A-04  

The `implementation-needs-checking` label means the SCT has not yet verified the implementations are complete and correct. Once the spec text is finalised (A-01–A-04), the SCT needs to do a spot-check of the Synapse + js-sdk implementations against the spec.

**What to do:** Once A-01–A-04 are done, request `implementation-needs-checking` review from @Half-Shot and @hughns (both are listed as awaiting review).

**Potential issue:** The `running_since` → `scheduled_ts` rename (A-02) will require all implementations to be updated before this sign-off can happen. If implementations are updated before the spec change lands, there may be a brief period where spec and code are out of sync — manage this by opening implementation PRs that are gated on the spec PR merging.

---

### A-06 · MSC4140 FCP countdown (5 SCT days)

**Status:** Pending  
**Owner:** SCT  
**Depends on:** A-05  

Once all blockers are clear and `implementation-needs-checking` is resolved, any SCT member proposes FCP. The 5-day countdown begins. Any SCT member can raise a concern to stop it.

**Risk:** Someone raises a new concern during FCP. Mitigated by the fact that @turt2live is already satisfied with the direction ("looking closer to being ready").

**What to watch:** No `unresolved-concerns` label on MSC4140 currently — this is good.

---

### A-07 · MSC4140 merge

**Status:** Pending  
**Depends on:** A-06  

After FCP completes with no blocking concerns, the MSC is merged. Synapse stable prefixes get enabled; js-sdk removes unstable prefix compatibility code.

**Post-merge action:** Synapse will need a follow-up release enabling stable endpoints. Clients need to switch from `/_matrix/client/unstable/org.matrix.msc4140/...` to `/_matrix/client/v3/...` prefixes. This must be coordinated across Synapse releases.

---

## Track B — MSC4354: Sticky Events

**Current status:** `unresolved-concerns` label from @richvdh. @turt2live approved. Comprehensive @Johennes review pass just completed (May 19–21, 2026). Sliding Sync split into MSC4480. Critical rust-sdk implementation gap.

> ⚠️ **This track has a hard unknown blocker (B-01) and a critical implementation gap (B-04). It carries the most risk for overall timeline.**

---

### B-01 · Resolve @richvdh's "requested changes" — PRIMARY BLOCKER

**Status:** Blocked — @richvdh has requested changes; specifics unknown from PR timeline  
**Owner:** @kegsay / @richvdh  
**Depends on:** Nothing (must be investigated first)  

@richvdh (Richard van der Hout, Element) has formally "requested changes" in the GitHub review, which prevents the `unresolved-concerns` label from being removed. The specific concerns are in the review thread but were not visible in the PR timeline.

**What to do:**
1. Read @richvdh's review comments in full.
2. Classify each concern: (a) wording fix, (b) design issue requiring spec change, (c) architectural concern requiring rethink.
3. For (a): address inline.
4. For (b)/(c): open a discussion thread and involve the broader MatrixRTC team.

**Tradeoff scenarios:**
- **Best case:** Concerns are wording/clarity issues → addressable in a week.
- **Medium case:** Concerns are design issues (e.g., forward extremity impact, equivocation handling) → require new spec text and possibly implementation changes. Could take 2–6 weeks.
- **Worst case:** Concerns require fundamental rethink of the sticky event model → could require a new MSC or major revision. Unlikely given @turt2live's approval, but possible.

**Workaround:** There is no workaround. The `unresolved-concerns` label must be cleared before FCP. However, if @richvdh's concerns are narrow and addressable without blocking other work, B-02 through B-05 can proceed in parallel.

**Who to contact:** @richvdh directly via the PR thread or Matrix room `#matrix-spec:matrix.org`.

---

### B-02 · Address @Johennes' review comments on MSC4354

**Status:** In progress — review conducted May 19–20, 2026; most are small  
**Owner:** @kegsay  
**Depends on:** Nothing (independent of B-01)  

@Johennes left ~10 review comments May 19–20, covering:
1. Link MSC3401 and MSC3757 in the motivation section.
2. Tie-breaking wording: "same event type and `sticky_key`" (not just `sticky_key`).
3. Duplicate phrase "this led to the prototype to the current proposal" → remove one occurrence.
4. MSC4268 (Sharing room keys) has since merged — update text from "would help" to "has alleviated this".
5. `send another sticky event with just content.sticky_key set` → "is an alternative way" not "are".
6. Forward extremities section: add link to Synapse dummy events threshold config.
7. Empty-leave events (`N` events for `N` users leaving) — @Johennes questions whether they appear once or N times in `/sync`; wording may need clarifying.

**What to do:** Address each comment. Most are single-line changes. The tie-breaking semantics comment (#2) may need a more careful read to ensure the spec text is unambiguous.

---

### B-03 · Clarify forward extremities impact in MSC4354

**Status:** Open — surfaced by @Johennes May 19, 2026  
**Owner:** @kegsay / server experts  
**Depends on:** Nothing  

When sticky events from an offline server arrive late (after they've expired), adding them to the DAG creates new forward extremities. More forward extremities = more state resolution work for the receiving server. The proposal offers two mitigations:
1. Servers MAY send dummy events to merge extremities (Synapse has `dummy_events_threshold` since 2019).
2. Servers MAY choose not to add expired sticky events to forward extremities (trade: reduces delivery guarantees).

The spec needs to make a clear normative recommendation. @Johennes asked for a link to the Synapse option.

**What to do:**
1. Add the Synapse config link.
2. Make the recommendation concrete: should SHOULD or MAY apply to each option?
3. Consider whether Complement tests cover the forward extremity scenario.

**Tradeoff:** Mandating dummy events adds server complexity. Allowing servers to skip adding old sticky events reduces DAG convergence. The current "MAY/MAY" stance is permissive but leaves interoperability undefined.

---

### B-04 · matrix-rust-sdk: Implement sticky events (MSC4354) for MatrixRTC — CRITICAL GAP

**Status:** Not started — no PR found  
**Owner:** Unknown — no assignee identified  
**Depends on:** Nothing (can start now using unstable prefix)  
**Blocking:** MSC4354 `implementation-needs-checking` label; Element X adoption  

This is the **largest single gap** in the entire Layer 0–1 plan. The rust-sdk has:
- ✅ MSC4143 membership format compat (PR #3500, June 2024 — small Ruma API change)
- ✅ MSC4140 delayed events (PR #3600, 2024)
- ❌ **MSC4354 sticky events — NOT implemented**

Without this, Element X continues using the old state event format for `m.rtc.member`, meaning:
- The `implementation-needs-checking` on MSC4354 cannot be cleared (the spec requires qualifying implementations — js-sdk exists, but SCT will want a second independent client implementation).
- Element X users don't benefit from the sticky events membership model.
- The rust-sdk MatrixRTC module remains on the old format indefinitely.

**What to do:**
1. Add `sticky` field support to Ruma's event types (or confirm Ruma already models it).
2. Update `matrix-sdk-base` room model to process the `/sync` `sticky.events` section.
3. Update the MatrixRTC module (`matrix-sdk` layer) to:
   - Send `m.rtc.member` as a sticky event (with `sticky_duration_ms` query parameter on PUT `/send`).
   - Read membership from `sticky.events` in sync response instead of / in addition to room state.
   - Handle the dual-mode transition (rooms may have both old state events and new sticky events during migration).
   - Implement `sticky_key` semantics for join/leave (using the key-value store addendum).
4. Add heartbeat integration with MSC4140 delayed events (the rust-sdk already has this, but it must be wired to the new sticky event send path).
5. Write tests: unit tests + Complement integration tests.

**Effort estimate:** Large — comparable to the original MatrixRTC membership implementation. The js-sdk PR (#5017) was 857 additions / 419 deletions with 69 commits. Estimate 2–4 weeks of focused engineering.

**Tradeoff — dual-mode transition:** During the rollout period, a server may have both old `org.matrix.msc3401.call.member` state events and new sticky events in the same room. Clients must handle both, or calls between old and new clients will break. The js-sdk handles this via `listenForStickyEvents` / `listenForMemberStateEvents` flags. The rust-sdk needs the same dual-mode logic.

**Tradeoff — Ruma types:** If Ruma doesn't yet model the `sticky` top-level field, a Ruma PR is a prerequisite. This could be a ~1-week blocker upstream.

**Risk:** Without an owner identified, this could slip indefinitely. The Element team is the most likely implementor, but it needs to be explicitly assigned.

---

### B-05 · MSC4480: Implement Sliding Sync extension for sticky events

**Status:** Spec just opened (May 20, 2026); Synapse server PR open (#19591, March 2026) with a known gap  
**Owner:** @reivilibre (Synapse), @Johennes (spec), client owner TBD  
**Depends on:** B-01 (can proceed in parallel for implementation, but spec needs MSC4354 stable first)  

MSC4480 is a new dependency that was split out from MSC4354 to keep each proposal focused. It defines how sticky events are delivered over Simplified Sliding Sync (MSC4186).

**Server side (Synapse):**
- PR #19591 is open (March 20, 2026, 20 commits). The implementation broadly works but has one confirmed gap:
  - **Gap #19662:** Sticky events that exist in a room are NOT sent down to newly joined sync sessions. A new client that connects after a call has started won't receive existing participant memberships until the next sticky event is received. This is a correctness bug — clients would incorrectly show 0 participants.
- PR #19591 needs to be merged and #19662 fixed before MSC4480 can be implemented.

**Client side:**
- No client implementation of MSC4480 exists yet.
- The js-sdk sticky events PR (#5017) was written for v2 `/sync`, not Sliding Sync (MSC4186). A separate SSS integration is needed.
- The rust-sdk's Sliding Sync integration would also need updating (depends on B-04 first).

**What to do:**
1. Merge synapse#19591 (server sliding sync support).
2. Fix synapse#19662 (newly joined rooms gap).
3. Implement MSC4480 client extension in js-sdk (new `sticky_events` section in SSS room response).
4. Implement in rust-sdk (after B-04).
5. Pass `implementation-needs-checking` for MSC4480.

**Tradeoff:** MSC4480 is `needs-implementation` and `kind:core`. The SCT will require implementations before it can enter FCP. If MSC4480 doesn't have implementations, it blocks MSC4354 from fully clearing the `implementation-needs-checking` gate — unless the SCT decides that SSS support can come post-merge (as a follow-on). This should be explicitly decided with the SCT.

**Critical question for SCT:** Can MSC4354 enter FCP and merge before MSC4480 has implementations, with MSC4480 as a follow-on? Or must both be simultaneously implementable? This should be asked in `#matrix-spec:matrix.org`.

---

### B-06 · MSC4354 FCP countdown (5 SCT days)

**Status:** Pending  
**Owner:** SCT  
**Depends on:** B-01 (hard), B-02, B-03, B-04, and a decision on B-05  

Once `unresolved-concerns` is cleared and `implementation-needs-checking` is satisfied, any SCT member proposes FCP.

**Risk:** If B-04 (rust-sdk) is not done, the SCT may refuse to start FCP on MSC4354 due to insufficient implementation breadth. The js-sdk implementation exists but is Element-only. The SCT typically wants at least one non-Element qualifying implementation. Trixnity (GitLab MR #687) may qualify as the second — worth confirming with the SCT.

---

### B-07 · MSC4354 merge

**Status:** Pending  
**Depends on:** B-06  

After FCP, MSC4354 merges. The Matrix spec gains a new primitive.

**Post-merge:** Synapse stable prefix routes become active. Client SDKs switch from unstable to stable `sticky` field handling. `org.matrix.msc3401.call.member` state events remain supported during transition period.

---

## Track C — MSC4143: MatrixRTC

**Current status:** Blocked on Track A + Track B. SCT 9-task checklist already created (May 19, 2026). No FCP tickyboxes yet.

---

### C-01 · End-to-end implementation testing with stable dependencies

**Status:** Cannot start until A-07 + B-07  
**Owner:** @toger5 + Element call/web/X teams  
**Depends on:** A-07 (MSC4140 merged), B-07 (MSC4354 merged)  

Once both dependencies have merged and their stable prefixes are enabled in Synapse, a full integration test must be run:
- Element Call ↔ Element X call using the new stable format end-to-end.
- Element Call ↔ another third-party client (e.g., Fractal or Trixnity-based).
- Multi-participant call with federation (calls across different homeservers).
- Crash recovery: client crashes mid-call → delayed event fires → other participants see leave event.
- Sliding Sync client connects mid-call → receives correct participant list.

**Tradeoff:** This testing can be done with unstable prefixes before A-07/B-07 (and is already happening informally). However, the SCT will want evidence of testing against the final stable spec, not just the unstable version. Element Call is already in production using MatrixRTC in unstable form — the work here is to document and verify that the stable implementation matches the spec.

---

### C-02 · Work through the 9-task SCT checklist for MSC4143

**Status:** Checklist created by @turt2live, May 19, 2026 — contents not publicly summarised  
**Owner:** @toger5 + SCT  
**Depends on:** C-01  

The standard MSC checklist typically covers:
1. Security considerations complete and reviewed
2. Privacy considerations complete
3. Alternatives section adequate
4. Unstable prefix documented
5. Backwards compatibility analysis done
6. Implementation evidence provided (qualifying implementations listed)
7. Spec text complete and unambiguous
8. Cross-references to dependencies correct and up to date
9. No outstanding concerns / FCP tickyboxes created

For MSC4143 specifically, items that need attention:
- **Open to-dos in the MSC** (discovery/negotiation of available applications; early media) — need to either be addressed or explicitly deferred to future MSCs.
- **Unstable prefix migration** — how do rooms with `org.matrix.msc3401.call.member` state events migrate? Must be documented.
- **Session history reconstruction** — the MSC relies on `origin_server_ts` ordering which is server-settable. The known limitation (no `received_server_ts`) should be documented as a known issue.
- **E2EE key rotation** — the per-participant key sharing mechanism needs a security review.

**What to do:** Obtain the full checklist from @turt2live's comment in the PR thread. Work through each item systematically.

---

### C-03 · Address any issues raised during checklist review

**Status:** Pending  
**Owner:** @toger5  
**Depends on:** C-02  

Checklist review will likely surface a handful of spec text issues. These are expected to be resolvable quickly given the MSC is already well-developed.

**Known potential issues:**
- The open to-dos ("discovery and negotiation of available applications") need to be explicitly deferred with a note, or removed. Leaving open to-dos in a merged spec is not acceptable.
- The message ordering issue (session history without `received_server_ts`) should be called out as a known limitation, pointing at MSC4445 / stitched ordering as future work.

---

### C-04 · MSC4143 FCP countdown (5 SCT days)

**Status:** Pending  
**Owner:** SCT  
**Depends on:** C-01, C-02, C-03  

**Risk:** The 5-day FCP for MSC4143 is the most likely to attract concerns from the wider Matrix community, given the scope of the proposal. A community review period should be communicated in `#matrix-spec:matrix.org` and ideally on the Matrix.org blog before FCP is proposed.

---

### C-05 · MSC4143 merge → Layer 1 complete 🎉

**Status:** Pending  
**Depends on:** C-04  

After FCP, MSC4143 merges. The Matrix spec formally includes MatrixRTC. Stable prefixes become the spec-defined names. The spec version bumps toward 2.0.

**Post-merge actions:**
- Synapse enables stable `/_matrix/client/v1/rtc/transports` endpoint.
- matrix-js-sdk removes unstable prefix aliases.
- matrix-rust-sdk removes unstable prefix aliases.
- MSC4195 (LiveKit transport), MSC4075 (ringing), MSC4196 (m.call) can now enter FCP.
- `org.matrix.msc3401` prefixes are formally deprecated (migration timeline to be set).

---

## Cross-Cutting Concerns

### XC-01 · Unstable → stable prefix migration strategy

All current deployments use `org.matrix.msc3401.call.member` (the old format) or `org.matrix.msc4143.rtc.member` (new unstable). When MSC4143 merges and `m.rtc.member` becomes stable:
- Some rooms will have old state events. Clients must handle both.
- No automatic migration: rooms don't get a "converter event".
- Recommended approach: dual-read (listen to both old state events and new sticky events) for at least 12 months post-merge, then drop `org.matrix.msc3401` support.
- This needs to be coordinated across all MatrixRTC client implementations.

**Tradeoff:** An indefinitely long compatibility window means more code complexity. A hard cutoff risks breaking older clients. The js-sdk dual-mode flag (`listenForStickyEvents` / `listenForMemberStateEvents`) is the right pattern.

---

### XC-02 · `implementation-needs-checking` breadth requirement

The SCT currently requires qualifying implementations before MSC can enter FCP. For MSC4354, the qualifying implementations are:
- **js-sdk** ✅ (Element-authored)
- **Synapse** ✅ (Element-authored)
- **Trixnity** ✅ (Connect2x/famedly-authored) — may count as the independent non-Element implementation
- **matrix-rust-sdk** ❌ (missing)

**Question for SCT:** Does Trixnity qualify as the "independent implementation" required for FCP, or must rust-sdk also be done? If Trixnity qualifies, the rust-sdk implementation (B-04) is still important for adoption but not a strict FCP gate.

---

### XC-03 · The dual `m.rtc.member` + state event transition in federated rooms

During the transition period, a federated room may have servers running different versions of Synapse and different client generations. Possible scenarios:
- **Old client (msc3401 state) ↔ New client (MSC4354 sticky):** Each client sees its own format; neither sees the other's participation. Call quality degrades silently.
- **Mixed Synapse versions:** An old Synapse doesn't understand sticky events; a new one does. Membership events from sticky-event clients won't appear in old Synapse's room state, potentially affecting auth.

**Mitigation:** MSC4354's forward-push semantics ensure sticky events are propagated to all participating servers, including those not directly connected. However, old Synapse versions that don't understand sticky events will ignore them in state resolution. This is acceptable — old servers won't break, they just won't show the new members. New clients connecting to rooms roomed by old servers should fall back to state events.

**Recommendation:** Document this in the MSC4354 spec under "Backwards Compatibility". Create a compatibility matrix table.

---

### XC-04 · The newly-joined-room sticky events gap (Synapse + MSC4480)

Synapse#19662 tracks a correctness bug: when a Sliding Sync client joins a room that already has active sticky events (e.g., an ongoing call), the server does not backfill those sticky events. The client would show 0 participants even though a call is in progress.

For v2 `/sync` this is less critical (timeline backfill covers it). For Sliding Sync clients (which increasingly include all Element X users), this is a user-visible bug: you join a room, see no ongoing call, but others are already in the call.

**Mitigation before fix:** Clients on Sliding Sync can fall back to reading room state for `org.matrix.msc3401.call.member` to detect ongoing calls. This is the existing behaviour anyway.

**Fix path:** synapse#19591 (open) + synapse#19662 (planned). These must both land before MSC4480 can claim a complete implementation.

---

## Ticket Summary

### Track A — MSC4140 (estimated 4–6 weeks)

| Ticket | Title | Status | Effort |
|--------|-------|--------|--------|
| A-01 | Merge MSC4478: Move use cases out | In progress | Small |
| A-02 | Rename `running_since` → `scheduled_ts` | Agreed, not done | Medium (impl changes) |
| A-03 | Resolve `M_MAX_DELAY_EXCEEDED` → `/capabilities` | Agreed in principle | Medium |
| A-04 | Address @turt2live + @Johennes review comments | In progress | Small |
| A-05 | SCT `implementation-needs-checking` sign-off | Pending A-01–04 | External dependency |
| A-06 | MSC4140 FCP countdown | Pending A-05 | External (5 days) |
| A-07 | MSC4140 merge | Pending A-06 | — |

### Track B — MSC4354 (estimated 8–16 weeks, depending on B-01)

| Ticket | Title | Status | Effort |
|--------|-------|--------|--------|
| B-01 | Resolve @richvdh's requested changes | **HARD BLOCKER** | Unknown |
| B-02 | Address @Johennes' review comments | In progress | Small |
| B-03 | Clarify forward extremities normative language | Open | Small |
| B-04 | rust-sdk: Implement MSC4354 sticky events | **Not started** | Large (2–4 weeks) |
| B-05 | MSC4480: Sliding Sync + sticky events (server + client) | In progress (server) | Large |
| B-06 | MSC4354 FCP countdown | Pending B-01–05 | External (5 days) |
| B-07 | MSC4354 merge | Pending B-06 | — |

### Track C — MSC4143 (estimated 4–8 weeks after A+B land)

| Ticket | Title | Status | Effort |
|--------|-------|--------|--------|
| C-01 | End-to-end implementation testing | Pending A-07 + B-07 | Medium |
| C-02 | Work through 9-task SCT checklist | Pending C-01 | Small–Medium |
| C-03 | Address checklist issues | Pending C-02 | Unknown |
| C-04 | MSC4143 FCP countdown | Pending C-03 | External (5 days) |
| C-05 | MSC4143 merge → Layer 1 complete | Pending C-04 | — |

### Cross-cutting

| Ticket | Title | Priority |
|--------|-------|----------|
| XC-01 | Document unstable → stable prefix migration strategy | High |
| XC-02 | Clarify `implementation-needs-checking` breadth with SCT | High (ask now) |
| XC-03 | Document federated transition scenarios in MSC4354 | Medium |
| XC-04 | Fix newly-joined-room sticky events gap (Synapse#19662) | High (UX correctness) |

---

## Critical Path

```
TODAY
 │
 ├── [PARALLEL] A-01 → A-02 → A-03 → A-04 ──► A-05 ──► A-06 ──► A-07 (MSC4140 ✓)
 │
 ├── [PARALLEL] B-01 (BLOCKER) ──► ...
 │            ↕ independent
 │            B-02, B-03 (spec wording, parallel with B-01)
 │            B-04 (rust-sdk, independent — START NOW)
 │            B-05 (SSS, partially in progress)
 │             └── all must complete ──► B-06 ──► B-07 (MSC4354 ✓)
 │
 └── After A-07 + B-07:
     C-01 → C-02 → C-03 → C-04 → C-05 (MSC4143 ✓ = Layer 1 complete)
```

**The only work that cannot start now is C-01.** All other tickets can begin immediately. The two largest risks are B-01 (unknown scope of @richvdh's concerns) and B-04 (no owner assigned for the rust-sdk implementation).
