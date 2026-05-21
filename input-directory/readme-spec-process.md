# README.md — MSC Process (matrix-spec-proposals)

**Source:** https://raw.githubusercontent.com/matrix-org/matrix-spec-proposals/main/README.md

## Key Points

### What Needs an MSC
- Changes requiring updates to a non-insignificant portion of Matrix implementations.
- Contentious changes where consensus is needed.
- Does NOT need MSC: typos, unambiguous clarifications, bug fixes.

### Process Summary

1. Write proposal in Markdown (use `proposals/0000-proposal-template.md` as guide).
2. Open a Draft PR to `matrix-org/matrix-spec-proposals` in `proposals/` directory.
3. MSC number = PR number (update document after opening).
4. Mark as Ready when complete; SCT applies labels.
5. Seek community review; implement proof-of-concept.
6. Ask SCT to review in `#sct-office:matrix.org`.
7. SCT proposes FCP (Final Comment Period) — 5 calendar-day countdown.
8. After FCP: PR merged → MSC accepted → stable identifiers usable.
9. Transcribe into `matrix-org/matrix-spec` (formal spec document).

### Unstable Prefixes

While MSC is not merged:
- Event types: `org.matrix.mscNNNN.<stable-name>`
- Endpoints: `/_matrix/client/unstable/org.matrix.mscNNNN/<endpoint>`
- Feature flags: `unstable_features: { "org.matrix.mscNNNN": true }`

After acceptance (before spec release):
- Can use stable identifiers.
- Can signal via `unstable_features: { "org.matrix.mscNNNN.stable": true }`.

### Room Versions

Changes to event format or auth rules require a new room version, which requires its own MSC (plus a curating MSC to make it a "real" room version).

### Closing / Postponing

- Author can close any time before FCP.
- SCT can propose FCP with disposition "close" (abandoned/rejected) or "postpone" (good idea, wrong time).
