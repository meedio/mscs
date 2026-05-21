# PR #3401: MSC3401 Native Group VoIP Signalling

**Source:** https://github.com/matrix-org/matrix-spec-proposals/pull/3401

## Metadata

- **State:** Open (largely superseded)
- **Author:** @ara4n (Matthew Hodgson, Element/Matrix.org)
- **Created:** September 19, 2021
- **Branch:** `matthew/group-voip` → `main`
- **Commits:** 24
- **Additions:** +310 lines
- **Rendered:** https://github.com/matrix-org/matrix-spec-proposals/blob/matthew/group-voip/proposals/3401-group-voip.md

## PR Description

> Obsoletes #2359

## Status

Open but largely superseded by MSC4143. Was the basis for early Element Call implementations using `org.matrix.msc3401.call` and `org.matrix.msc3401.call.member` unstable prefixes.

MSC4143 explicitly plans to include a full-mesh WebRTC transport based on MSC3401 as a future transport option.

## Historical Significance

- First formal group VoIP MSC for Matrix
- Introduced concept of per-user state events for call membership
- Split: MSC3898 extracted from it to handle SFU cascading separately
- Drove the need for MSC4140 (delayed events) to handle crash membership cleanup
- Architecture problems (array membership, state resolution) led directly to MSC4143 + MSC4354 redesign
