# Gaps — Unknown / Unresolved Information

This file documents gaps in our initial research. Items marked **RESOLVED** have been answered through deep research. Items still marked **OPEN** require direct outreach.

*Last updated: May 21, 2026 — deep research pass completed.*

---

## Critical Gaps (blocking understanding)

### GAP-1: What are the "unresolved concerns" in MSC4354 (Sticky Events)?

**Status: PARTIALLY RESOLVED — active review happening RIGHT NOW**

As of May 19–21, 2026 (literally 1–2 days ago), MSC4354 is under the most intense review activity since it opened:

- **@turt2live approved** the proposal on May 19, 2026.
- **@richvdh has "requested changes"** — this is the main outstanding blocker. The specific concerns from @richvdh are in the PR thread but not summarised; they appear to relate to server federation semantics. This is the unresolved-concerns label source.
- **@Johennes** did a comprehensive review pass on May 19–20, 2026 with many suggestions (mostly spec wording, linkification, and a forward-extremities clarification).
- **Sliding Sync integration was split out**: @Johennes created **MSC4480** "Sliding Sync Extension: Sticky Events" (May 20, 2026), removing that portion from MSC4354 to simplify it.
- The proposal text was **updated on May 20, 2026** to remove the Sliding Sync section.
- A Trixnity implementation exists (GitLab MR #687), and a Dart SDK implementation is in progress (matrix-dart-sdk#2238, opened April 27, 2026).
- Synapse 1.148.0 (Feb 2026) ships sticky events support.

**What remains open:**
1. @richvdh's "requested changes" — specific concerns in the PR thread, not publicly summarised.
2. Some open @Johennes review comments being addressed by @kegsay (tie-breaking semantics in key-value store, forward extremities wording).
3. MSC4480 (Sliding Sync extension) is now a dependency, though it was just created.

**Assessment:** The `unresolved-concerns` label is primarily about @richvdh's requested changes. Resolution appears close given the current pace of review.

**Who to ask:** @richvdh (Richard van der Hout) directly via the PR thread. Also @kegsay.

**Where:** https://github.com/matrix-org/matrix-spec-proposals/pull/4354

---

### GAP-2: What are the three specific implementation gaps in MSC4140 (Delayed Events)?

**Status: RESOLVED — gaps were removed, MSC is "Proposed for FCP readiness"**

The three implementation gap commits listed in the old PR description have been **removed** as of May 20, 2026 via the commit "Remove implementation TODOs — will be tracked in MSC comments instead". They are no longer formal blockers listed in the PR description.

The Synapse implementation has expanded significantly and now includes 5 merged PRs:
- `element-hq/synapse#17326` — original delayed events support
- `element-hq/synapse#19038` — finalised delayed events and more
- `element-hq/synapse#19354` — dedicated endpoint for delayed events
- `element-hq/synapse#19479` — `delay_id` in unsigned data for sender
- `element-hq/synapse#19539` — update error responses

The JS-SDK has 3 merged PRs and Ruma/matrix-rust-sdk are also implemented.

**Current status (as of May 2026):**
- **Project status: "Proposed for FCP readiness"** — the SCT is actively moving this toward FCP.
- @turt2live said on May 19, 2026: *"this is looking closer to being ready for FCP"*.
- @Johennes did a comprehensive review (May 18–20, 2026) with many wording suggestions being addressed.
- **MSC4479** "Shorten alternatives" was **merged** on May 20, 2026.
- **MSC4478** "Move use cases section out" is being created (in progress).

**Remaining open issues (spec wording, not blockers):**
- `M_INVALID_PARAM` vs `M_MAX_DELAY_EXCEEDED` error code — likely to use `/capabilities` for max delay instead.
- `running_since` field naming — likely rename to `scheduled_ts`.
- `delay_id` vs `delay_token` naming — @AndrewFerr argues against rename; @toger5 open to it.

**Assessment:** This MSC will likely enter FCP in Q2/Q3 2026. The original "3 gaps" are no longer the blocking issue — spec language review is the current work.

---

### GAP-3: Is there an active timeline or roadmap for MSC4354 + MSC4140 → MSC4143 acceptance?

**Status: RESOLVED — active movement happening in May 2026**

Evidence that these MSCs are being actively driven toward acceptance:

1. **Matrix 2.0 was declared "here"** at The Matrix Conference in October 2024 (blog post: https://matrix.org/blog/2024/10/29/matrix-2.0-is-here/). MatrixRTC is one of the four core pillars. The spec version bump to 2.0 will happen when the MSCs pass FCP.

2. **MSC4140 is "Proposed for FCP readiness"** in the SCT project board as of May 2026. Active review is happening this week.

3. **MSC4354 is under intense review** from @Johennes and @turt2live this week (May 19–21, 2026). @turt2live approved it on May 19.

4. **MSC4143 received a 9-task SCT checklist** on May 19, 2026 (created by @turt2live, visible as a comment in the PR).

5. **No hard deadline** is publicly stated, but the pace of activity in May 2026 suggests FCP for MSC4140 and MSC4354 in Q2–Q3 2026, followed by MSC4143 shortly after.

**Assessment:** The path is: MSC4140 FCP → merge, MSC4354 FCP (pending @richvdh's concerns) → merge, then MSC4143 implementation testing + FCP. This could realistically happen in 2026 H2 if the current pace holds.

---

### GAP-4: What is the current state of MSC4075 (RTC Notifications / Ringing)?

**Status: RESOLVED — substantially more detailed picture obtained**

| Field | Value |
|-------|-------|
| **Author** | @toger5 (Timo K) |
| **Created** | November 8, 2023 |
| **Last major update** | July 2025 (renamed `m.rtc.notification`, added `lifetime` + `m.reference` relation) |
| **Commits** | 19 |
| **Additions** | +227 lines |
| **Branch** | `toger5/matrixrtc-call-ringing` |
| **Labels** | `kind:feature`, `matrix-2.0`, `needs-implementation`, `proposal`, `voip` |
| **SCT Project Status** | "Tracking for review" |

**What it defines:**
- `m.rtc.notification` event for ringing/notifying before someone joins a MatrixRTC slot
- `notification_type`: `"ring"` | `"notification"`
- `lifetime` field: milliseconds the receiver should ring before auto-stopping
- `m.relates_to` with `m.reference` pointing to the caller's `m.rtc.member` event
- `m.call.intent` field (`"voice"` | `"video"`) — already in production implementations

**Implementations:**
- Element Web (matrix-react-sdk#11870) ✅
- Ruma (ruma#1704, updated ruma#2199) ✅
- Element X Android: ✅ (MSC-updated format)
- Element X iOS: partial
- matrix-js-sdk#4826 (revised format) — in progress

**Open issues:**
- `m.call.intent` field is in all implementations but needs to be formalised in the proposal text
- Timeline rendering: @toger5's position is that `m.rtc.notification` should NOT render in the timeline — historic calls should be reconstructed from `m.rtc.member` session overlaps. There is debate on this.
- How to decline a call → handled by **MSC4310** (`m.rtc.decline`) — created July 2025, has implementations in matrix-rust-sdk and Ruma.

**Rendered proposal:** https://github.com/matrix-org/matrix-spec-proposals/blob/toger5/matrixrtc-call-ringing/proposals/4075-rtc-notification-event.md

---

### GAP-5: What is the full proposal text of MSC4196 (m.call application)?

**Status: RESOLVED — full current text obtained**

The January 12, 2026 rewrite by @fkwp (Florian Kaltenberger) clarifies the `m.call` application type. The open to-dos from our initial research have been answered:

**Voice vs. video distinction:** Resolved via `m.call.intent` field:
```json
"application": {
  "type": "m.call",
  "m.call.intent": "voice" | "video"  // optional informational hint
}
```
This field is **informational only** — non-authoritative, just a hint about the session type.

**Slot model:** Only one slot type defined: `m.call#ROOM` — a room-level slot open to all members with sufficient power level. Suitable for DMs (telephone-style) and group calls.

**`trusted_private_chat` preset** should auto-enable a default `m.call#ROOM` slot.

**For ringing**, extends `m.rtc.notification` (MSC4075) with `m.call.intent`.

**Status:** Still **Draft**, `needs-implementation` label. Depends on MSC4143 and MSC4075.
**Author:** @hughns (initial), @fkwp (Jan 2026 rewrite).
**Rendered:** https://github.com/matrix-org/matrix-spec-proposals/blob/hughns/matrixrtc-m-call/proposals/4196-matrixrtc-m-call.md

---

## Secondary Gaps (important but not immediately blocking)

### GAP-6: What is the plan for the full-mesh WebRTC transport (MSC3401 within MSC4143)?

**Status: PARTIALLY RESOLVED**

From the Matrix 2.0 blog post (October 2024):
> "today, the main implementation uses the LiveKit SFU, but there's also an **experimental full-mesh WebRTC implementation**."

So there IS an experimental full-mesh implementation in Element Call, but it operates using the unstable MSC3401 prefixes and hasn't been formally packaged as a MatrixRTC transport MSC.

No formal MSC for the full-mesh transport under the MSC4143 framework has been opened as of May 2026. The Element team is focused on the LiveKit path. The full-mesh transport MSC remains future work with no stated owner or timeline.

**Remaining open question:** Is anyone planning to formally write the full-mesh transport MSC for MSC4143, or will the experimental implementation remain unofficial?

**Who to ask:** @toger5, @ara4n.

---

### GAP-7: What is the current status of MSC3898 (Cascading SFU)?

**Status: CONFIRMED DORMANT** — no new information found. Last activity January 2023.

The federated multi-SFU approach in MSC4195 has effectively superseded MSC3898. No formal close/postpone has been proposed. It continues to exist as an open WIP PR with no active work.

---

### GAP-8: What happens to `org.matrix.msc3401` unstable prefixes already deployed?

**Status: PARTIALLY RESOLVED — migration is organic/in-flight**

From the Matrix 2.0 blog post (Oct 2024), Element Call and Element X already use MatrixRTC (the new system) in production. The migration is happening through client updates rather than a formal migration MSC.

Element X uses `org.matrix.msc4143.rtc.member` unstable prefixes in the new format. Element Web/Desktop runs Element Call in embedded mode, which handles both old and new formats.

**No formal migration MSC or guide** has been written as of May 2026. Clients that supported MSC3401 need to handle both old and new event formats during the transition.

**Remaining open question:** When will MSC3401-format events be formally deprecated, and is there a target end-of-life date?

**Who to ask:** @toger5, @hughns.

---

### GAP-9: Is MSC4309 (Delayed Event on Sync) being actively developed?

**Status: UNRESOLVED** — No new information found.

MSC4309 remains an early draft from August 2025 with no recent activity. It is likely **not** a blocker for MSC4140 FCP — it appears to be a post-merge improvement. The SCT has not listed it as a dependency.

**Who to ask:** @toger5 (likely author).

---

### GAP-10: How does session history work without `received_server_ts`?

**Status: PARTIALLY RESOLVED — related work found, no `received_server_ts` MSC**

Two related efforts found:

1. **MSC4445: Clarify `/sync` timeline order** (April 1, 2026, by @MadLittleMods) — a `kind:maintenance` MSC that clarifies how events should be ordered in `/sync` responses. It does not introduce `received_server_ts` but addresses related ordering ambiguity. @andybalaam (stitched ordering author) reviewed it. Status: `needs-implementation`, active review from @turt2live.

2. **Stitched ordering** (https://codeberg.org/andybalaam/stitched-order) — @andybalaam's research project into deterministic event ordering algorithms. Referenced in MSC4143 as a potential solution but not an MSC yet.

**No MSC for `received_server_ts`** has been opened. MSC4143's workaround (`origin_server_ts` ordering) remains the current approach. MSC4143 explicitly notes this as an open problem.

**Remaining open question:** Is there a plan to open an MSC for `received_server_ts` or stitched ordering? When would this matter — only after MSC4143 merges?

**Who to ask:** @andybalaam, @toger5.

---

### GAP-11: What is the SCT checklist for MSC4143?

**Status: RESOLVED — checklist exists as of May 19, 2026**

@turt2live created a **9-task SCT checklist** for MSC4143 in a PR comment on May 19, 2026. This is referenced from both the MSC4140 and MSC4354 PRs as "MSC4143: MatrixRTC — 9 tasks". The PR description itself says "See #4143 (comment) for further status information."

The PR description still lists:
- **BLOCKED** on MSC4354 and MSC4140 landing
- No FCP tickyboxes in the description yet (those would come once dependencies land)

**Assessment:** The SCT is already preparing for MSC4143 review (creating the checklist). Once MSC4354 and MSC4140 merge, implementation testing begins, then the 9-task checklist will be worked through before FCP can be proposed.

The 9 tasks likely include the standard MSC checklist items: security considerations, alternatives, unstable prefixes, implementation evidence, etc. The full list is in a comment in https://github.com/matrix-org/matrix-spec-proposals/pull/4143 (not visible in the PR timeline view).

---

## New MSCs Discovered During Research

These MSCs were not in our initial catalog but are relevant:

| MSC | Title | Status | Role |
|-----|-------|--------|------|
| [MSC4310](https://github.com/matrix-org/matrix-spec-proposals/pull/4310) | MatrixRTC decline `m.rtc.decline` | Open | How to decline a call (dependency of MSC4075) |
| [MSC4445](https://github.com/matrix-org/matrix-spec-proposals/pull/4445) | Clarify `/sync` timeline order | Open | Addresses event ordering (related to session history in MSC4143) |
| [MSC4478](https://github.com/matrix-org/matrix-spec-proposals/pull/4478) | \*MSC4140: Move use cases section out | In progress (May 2026) | Amendment to MSC4140 to slim it down |
| [MSC4479](https://github.com/matrix-org/matrix-spec-proposals/pull/4479) | \*MSC4140: Shorten alternatives | **Merged** May 20, 2026 | Amendment to MSC4140 — shorter alternatives section |
| [MSC4480](https://github.com/matrix-org/matrix-spec-proposals/pull/4480) | Sliding Sync Extension: Sticky Events | Open (May 20, 2026) | Split from MSC4354 to handle Sliding Sync integration separately |
| [MSC4471](https://github.com/matrix-org/matrix-spec-proposals/pull/4471) | Streaming ephemeral event updates for room events | Open (May 19, 2026) | New MSC opened by @kegsay (MSC4354 author) — possibly related |

---

## Questions for Direct Outreach (Updated Priority)

1. **"What specifically are @richvdh's requested changes in MSC4354?"** — This is now the primary blocker. (@richvdh directly, or @kegsay)
2. **"Is MSC4140 expected to enter FCP before MSC4354, or are they being driven in parallel?"** (@turt2live / SCT)
3. **"What are the 9 tasks in the MSC4143 SCT checklist?"** (@turt2live)
4. **"When will `m.call.intent` be formally added to the MSC4075 proposal text?"** (@toger5)
5. **"Is there a plan to write a full-mesh WebRTC transport MSC for MSC4143?"** (@ara4n / @toger5)
6. **"What is the migration timeline for deprecated `org.matrix.msc3401` event types?"** (@toger5 / @hughns)
