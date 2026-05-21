# PR #4075: MSC4075 MatrixRTC m.rtc.notification (Call Ringing)

**Source:** https://github.com/matrix-org/matrix-spec-proposals/pull/4075

## Metadata

- **State:** Open
- **Labels:** voip, matrix-2.0
- **Dependencies:** MSC4143

## Notes

Defines pre-call signalling (ringing, decline, busy) for MatrixRTC using `m.rtc.notification`. Fills the gap between MSC4143 (in-session) and the calling UX users expect. Required by MSC4196 for the full `m.call` experience.
