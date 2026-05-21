# Gaps — Unknown / Unresolved Information

This file documents information that is **not clearly stated in any public document** and represents gaps that must be filled — ideally by asking the authors directly — before we can fully understand the current state of the MatrixRTC MSC ecosystem.

---

## Critical Gaps (blocking understanding)

### GAP-1: What are the "unresolved concerns" in MSC4354 (Sticky Events)?

**Why it matters:** MSC4354 has the `unresolved-concerns` label and FCP tickyboxes have been created, but the specific open concerns are not summarised anywhere obvious. Sticky Events is the **primary blocker** for MSC4143.

**What we need:** A list of the specific open concerns and their status (resolved / in-progress / blocked on design decision). Are these architectural? Implementation details? Federation edge cases?

**Who to ask:** @kegsay (Kegan Dougal) — author of MSC4354. Also @ara4n, @turt2live, @erikjohnston who reviewed.

**Where:** Comment thread in https://github.com/matrix-org/matrix-spec-proposals/pull/4354

---

### GAP-2: What are the three specific implementation gaps in MSC4140 (Delayed Events)?

**Why it matters:** MSC4140 has three known unresolved implementation gaps (commits `3ef314f`, `95045cf`, `49b200d`) that are blocking FCP. MSC4140 must land before MSC4143 can proceed.

**What we need:** What exactly are these gaps? Are they server-side (Synapse), client-side, or spec wording? Is anyone actively working on closing them? What is the estimated timeline?

**Who to ask:** @toger5 (Timo K) — author of both MSC4140 and MSC4143.

**Where:** https://github.com/matrix-org/matrix-spec-proposals/pull/4140

---

### GAP-3: Is there an active timeline or roadmap for MSC4354 + MSC4140 → MSC4143 acceptance?

**Why it matters:** There is no public roadmap. All three MSCs are open with no FCP started. Understanding whether there is a target milestone (e.g. Matrix v1.X or a specific Element release cycle) would clarify how far we are from a stable MatrixRTC spec.

**What we need:** Is the SCT actively prioritising these MSCs? Is there a Matrix 2.0 launch target date? Are MSC4354/MSC4140 being fast-tracked?

**Who to ask:** @turt2live (Travis Ralston) — SCT member who manages labels and FCP on all three MSCs. Also @richvdh (Richard van der Hout) or other SCT members.

**Where:** #sct-office:matrix.org, or directly via the MSC PR threads.

---

### GAP-4: What is the current state of MSC4075 (RTC Notifications / Ringing)?

**Why it matters:** MSC4075 is required by MSC4196 (m.call application) to provide a complete calling experience with ringing. We have very little detail on its content, current state, or who is actively working on it.

**What we need:** Full proposal text, current review status, known issues, author, timeline.

**Who to ask:** Element VoIP team — likely same people as MSC4143/MSC4196 (@toger5, @hughns).

**Where:** https://github.com/matrix-org/matrix-spec-proposals/pull/4075

---

### GAP-5: What is the full proposal text of MSC4196 (m.call application)?

**Why it matters:** MSC4196 defines the complete `m.call` voice/video experience. We know it was significantly rewritten in January 2026 but do not have the current text cached. The open to-dos (voice vs video distinction, field structures) need to be fully understood.

**What we need:** The current rendered proposal text.

**Where:** https://github.com/matrix-org/matrix-spec-proposals/pull/4196

---

## Secondary Gaps (important but not immediately blocking)

### GAP-6: What is the plan for the full-mesh WebRTC transport (MSC3401 within MSC4143)?

**Why it matters:** MSC4143 mentions a full-mesh transport based on MSC3401 as a "planned" transport, but no MSC for this transport has been opened. It is unclear if anyone intends to write it, or if the SFU-first approach (MSC4195) is the only planned transport for the foreseeable future.

**What we need:** Is anyone planning to write a full-mesh transport MSC for MSC4143? Is MSC3401 being formally retired or just deprioritised?

**Who to ask:** @ara4n (Matthew Hodgson) and @toger5.

---

### GAP-7: What is the current status of MSC3898 (Cascading SFU)?

**Why it matters:** MSC3898 has been dormant since January 2023. Its multi-SFU problem is effectively addressed by MSC4195 instead. But its formal disposition (close vs supersede vs retain) is unclear.

**What we need:** Is MSC3898 going to be formally closed? Or is the cascading approach still relevant for some use cases (e.g. very large federated calls)?

**Who to ask:** @SimonBrandner (author), or @ara4n and the Element VoIP team.

---

### GAP-8: What happens to `org.matrix.msc3401` unstable prefixes already deployed?

**Why it matters:** Many production deployments of Element Call use `org.matrix.msc3401.call` and `org.matrix.msc3401.call.member` unstable event types. The migration path to MSC4143's `m.rtc.slot` / `m.rtc.member` is not documented.

**What we need:** Is there a migration plan/guide? Will there be a bridge period where both are supported? Is this handled in MSC4143 or a separate migration MSC?

**Who to ask:** @toger5, @hughns.

---

### GAP-9: Is MSC4309 (Delayed Event on Sync) being actively developed?

**Why it matters:** MSC4309 is a `needs-implementation` draft from August 2025 with no further updates. The heartbeat reliability story for MatrixRTC is incomplete without it.

**What we need:** Is this on anyone's roadmap? Is it considered a blocker for MSC4140 FCP or is it a post-merge improvement?

**Who to ask:** @toger5 (likely same author given the context).

---

### GAP-10: How does session history work without `received_server_ts`?

**Why it matters:** MSC4143 acknowledges that accurate session history reconstruction requires consistent message ordering, ideally using `received_server_ts`. This field doesn't exist today. The current workaround uses `origin_server_ts` which can be unreliable under federation lag.

**What we need:** Is there a planned MSC or proposal to add `received_server_ts`? The MSC references "[stitched ordering](https://codeberg.org/andybalaam/stitched-order/src/branch/main/msc/msc.md)" — what is the status of this work?

**Who to ask:** @andybalaam (Andy Balaam, who wrote the stitched ordering proposal), @toger5.

---

### GAP-11: What is the SCT checklist for MSC4143?

**Why it matters:** The PR description states "No MSC checklist" and "No FCP tickyboxes" for MSC4143. These need to be created before FCP can be proposed. Understanding what items are on the checklist tells us how much review work remains.

**What we need:** Has any SCT member started reviewing MSC4143 in detail? Are there known concerns beyond the dependency blockers?

**Who to ask:** Any SCT member — @turt2live, @richvdh, @ara4n.

---

## Questions for Direct Outreach

If reaching out to the MSC authors directly (e.g. in `#sct-office:matrix.org` or `#matrix-spec:matrix.org`), these are the highest-value questions:

1. **"What specifically are the unresolved concerns in MSC4354, and what is needed to resolve them?"** (to @kegsay)
2. **"What are the three implementation gaps in MSC4140, and are they being actively worked on?"** (to @toger5)
3. **"Is there a target timeline or milestone for the MSC4354 + MSC4140 + MSC4143 stack to reach FCP?"** (to @turt2live / SCT)
4. **"Is the plan still to write a full-mesh WebRTC transport MSC based on MSC3401, or is MSC4195/LiveKit the only planned transport?"** (to @ara4n / @toger5)
5. **"What is the migration path from `org.matrix.msc3401.call` to `m.rtc.slot`/`m.rtc.member`?"** (to @toger5 / @hughns)
