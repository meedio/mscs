# PR #4196: MSC4196 MatrixRTC m.call Application

**Source:** https://github.com/matrix-org/matrix-spec-proposals/pull/4196

## Metadata

- **State:** Open (draft)
- **Author:** Element team
- **Created:** September 19, 2024
- **Last updated:** January 12, 2026
- **Labels:** matrix-2.0, voip, proposal
- **Dependencies:** MSC4143, MSC4075

## Notes

- Major rewrite on January 12, 2026 addressing feedback from offline discussions
- Defines `m.call` application type for voice/video conferencing
- Sets schema for `m.rtc.slot` and `m.rtc.member` event content for calls
- Open to-dos: distinguishing voice vs video calls, finalizing field structures
- Cinny client added video call support referencing this MSC (Feb 2026)
- Blocked on MSC4143 (and MSC4075 for full ringing support)
