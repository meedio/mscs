# Matrix Spec Change (MSC) Lifecycle

## What is an MSC?

A Matrix Spec Change (MSC) is a formal technical proposal for a change to the [Matrix Protocol](https://spec.matrix.org). MSCs live in the [`matrix-org/matrix-spec-proposals`](https://github.com/matrix-org/matrix-spec-proposals) repository as GitHub Pull Requests, and the proposal document is a Markdown file added to the `proposals/` directory.

Changes that require an MSC are ones that would need updates to a non-insignificant portion of the Matrix implementation ecosystem, or that would be met with contention. Bug fixes, typos, and unambiguous clarifications to existing behaviour generally do not need an MSC.

---

## The Five-Stage Lifecycle

### Stage 1: Drafting

The author writes a proposal Markdown document. The proposal must:
- Unambiguously describe the technical change (endpoints, event types, schemas, edge cases).
- Justify *why* the change benefits the whole ecosystem, not just one party.
- Not include patent-encumbered IP.
- Follow the [DCO sign-off requirement](#contributing--sign-off).

A [proposal template](https://github.com/matrix-org/matrix-spec-proposals/blob/main/proposals/0000-proposal-template.md) is available but not mandatory. The MSC number is the GitHub PR number, which is not known before opening the PR, so authors use a placeholder (`0000` or `XXXX`) and update it after submission.

### Stage 2: Open PR (Community Review)

1. Author opens a Pull Request to `matrix-org/matrix-spec-proposals` adding the proposal file to `proposals/`.
2. PR is **marked as Draft** until it is ready for wider review.
3. The Spec Core Team (SCT) applies labels (e.g. `proposal`, `kind:feature`, `client-server`, `matrix-2.0`, `needs-implementation`).
4. Community reviews the PR: anyone can leave comments, ask questions, or approve. Getting review from people familiar with the relevant area is especially valuable.
5. A **proof-of-concept implementation** in at least one client or server is expected before the MSC can be accepted.
6. The PR must use **unstable prefixes** for all new identifiers (event types, endpoint paths, field names) until the MSC is merged. Convention: `org.matrix.mscNNNN.<stable-name>` or `/_matrix/client/unstable/org.matrix.mscNNNN/<endpoint>`.
7. Implementations can advertise support via `unstable_features` in `/_matrix/client/versions`.

### Stage 3: Final Comment Period (FCP)

1. The author asks a member of the SCT to review the proposal in the [Office of the SCT Matrix room](https://matrix.to/#/#sct-office:matrix.org).
2. An SCT member reviews and, if satisfied, **proposes FCP** (Final Comment Period).
3. Other SCT members vote on the proposal.
4. Once enough SCT members approve, the MSC enters a **5 calendar-day countdown** during which anyone can raise final blockers.
5. FCP can be proposed with three dispositions:
   - **accept** — merge the MSC.
   - **close** — reject the MSC (e.g. abandoned or widely rejected).
   - **postpone** — good idea but not appropriate for the current ecosystem state.

### Stage 4: Accepted (Merged)

- The PR is merged and the MSC is **officially part of the Matrix Spec**.
- Implementations may now use **stable identifiers** (`m.<name>`, `/_matrix/client/v1/<endpoint>`).
- The homeserver can signal stable support via `unstable_features: { "org.matrix.mscNNNN.stable": true }`.
- The change still needs to be **transcribed into the formal spec** at [`matrix-org/matrix-spec`](https://github.com/matrix-org/matrix-spec). Authors are welcome to do this themselves; otherwise it is handled by an SCT member.

### Stage 5: Spec Release

- The next spec release (e.g. Matrix v1.X) incorporates the merged MSC text.
- Homeservers that support that spec version advertise it under `/_matrix/client/versions`.

---

## Lifecycle Summary Diagram

```
 Author writes proposal
        │
        ▼
  Opens Draft PR  ──── community review, iterations ────►  Ready
        │
        ▼
  Out of Draft (Open PR, labels applied by SCT)
        │
        ▼
  Author requests SCT review in #sct-office:matrix.org
        │
        ▼
  SCT member reviews → proposes FCP
        │
        ▼
  FCP vote by SCT members
        │
  ┌─────┴──────────────────────┐
  ▼                            ▼
accept (5-day countdown)    close / postpone
  │
  ▼
PR Merged (MSC Accepted)
  │
  ▼
Stable identifiers usable
  │
  ▼
Transcribed into matrix-spec
  │
  ▼
Next spec release (Matrix v1.X)
```

---

## Common PR Labels

| Label | Meaning |
|-------|---------|
| `proposal` | This PR is a formal MSC |
| `kind:feature` | Proposes a new capability |
| `kind:core` | Fundamental change to the protocol |
| `kind:maintenance` | Maintenance fix to an existing MSC |
| `client-server` | Affects the Client-Server API |
| `server-server` | Affects the Federation API |
| `needs-implementation` | Must be implemented before FCP |
| `implementation-needs-checking` | Implementations exist but need review |
| `unresolved-concerns` | Outstanding open issues blocking FCP |
| `matrix-2.0` | Part of the Matrix 2.0 initiative |
| `voip` | Relates to VoIP/RTC functionality |
| `blocked` | Cannot proceed until a dependency lands |
| `abandoned` | Author is no longer maintaining this |

---

## Contributing / Sign-off

All contributions must be signed off using the **Developer Certificate of Origin (DCO)**. Include the following line in your commit or PR comment:

```
Signed-off-by: Your Name <your@email.example.org>
```

Or use `git commit -s` to add it automatically. The project is licensed under **Apache Software License v2**.

---

## Where to Get Help

| Room | Purpose |
|------|---------|
| [#matrix-spec:matrix.org](https://matrix.to/#/#matrix-spec:matrix.org) | General MSC/spec discussion |
| [#sct-office:matrix.org](https://matrix.to/#/#sct-office:matrix.org) | SCT review requests; high signal, low noise |
| [#matrix-spec-process:matrix.org](https://matrix.to/#/#matrix-spec-process:matrix.org) | Questions about the MSC process itself |
| [#matrix-docs:matrix.org](https://matrix.to/#/#matrix-docs:matrix.org) | Formal spec text and matrix.org website |

---

## Key Resources

- [MSC listing (stable)](https://spec.matrix.org/proposals)
- [MSC listing (unstable)](https://spec.matrix.org/unstable/proposals/)
- [matrix-spec-proposals GitHub repo](https://github.com/matrix-org/matrix-spec-proposals)
- [Contributing guide (CONTRIBUTING.md)](https://github.com/matrix-org/matrix-spec-proposals/blob/main/CONTRIBUTING.md)
- [MSC Checklist](https://github.com/matrix-org/matrix-spec-proposals/blob/main/MSC_CHECKLIST.md)
- [Proposal template](https://github.com/matrix-org/matrix-spec-proposals/blob/main/proposals/0000-proposal-template.md)
- [Spec Core Team](https://matrix.org/foundation/about/#the-spec-core-team)
