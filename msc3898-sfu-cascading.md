# MSC3898: Native Matrix VoIP Signalling for Cascaded Foci (SFUs, MCUs...)

## Summary

MSC3898 proposes **cascaded SFU/MCU signalling** for Matrix group calls. It extends the `m.call.member` state event (introduced in MSC3401) with two optional fields that allow participants to advertise which SFU they are currently connected to and which SFU they would prefer to use. This enables geographic routing optimisation and federated multi-SFU topologies in a full-mesh-style arrangement.

> **Note:** This MSC addresses cascading/SFU federation in the legacy MSC3401 framework. The MSC4195 LiveKit transport (under MSC4143) takes a different approach to multi-SFU that does not require cascading — each participant connects to their own homeserver's SFU directly.

---

## Status

| Field | Value |
|-------|-------|
| **PR** | [#3898](https://github.com/matrix-org/matrix-spec-proposals/pull/3898) |
| **State** | Open (WIP / largely dormant) |
| **Author** | [@SimonBrandner](https://github.com/SimonBrandner) |
| **Created** | September 25, 2022 |
| **Last updated** | January 3, 2023 |
| **Labels** | `needs-implementation` |
| **FCP** | Not started |
| **Lifecycle stage** | Draft / stalled — activity has moved to MSC4195 |

---

## Proposal

Two optional fields are added to the `m.call.member` state event (MSC3401):

### `m.foci.active`

A list of foci (SFU/MCU servers) the participant's device is **currently connected to and publishing media to**. Normally contains one entry, but may contain multiple in edge cases (e.g. simultaneous publication to multiple networks).

```json
{
  "m.foci.active": [
    {
      "type": "livekit",
      "livekit_service_url": "https://livekit.example.com"
    }
  ]
}
```

### `m.foci.preferred`

A list of foci the participant **would prefer to switch to**, if other participants start using them. This allows the system to converge on a geographically optimal SFU.

```json
{
  "m.foci.preferred": [
    {
      "type": "livekit",
      "livekit_service_url": "https://livekit.eu.example.com"
    }
  ]
}
```

### Problem Solved

Without cascading, all participants in a geographically distributed call would need to connect to the same SFU, potentially a distant one, wasting bandwidth. With `m.foci.preferred`, participants can signal their local SFU preference, and when enough participants share that preference, others can switch.

---

## Why This MSC Is Largely Dormant

The activity around multi-SFU federation has moved to the MSC4143 / MSC4195 architecture:

- MSC4195 uses a **federated multi-SFU model by default**: each participant publishes to their homeserver's SFU, eliminating the need for SFU election entirely.
- MSC3898 was designed for the MSC3401 architecture which has itself been superseded.
- There is no active implementation work on MSC3898 as of early 2026.

The `needs-implementation` label has been present since creation and no implementations have been reported.

---

## Relationship to Other MSCs

| MSC | Relationship |
|-----|-------------|
| [MSC3401](https://github.com/matrix-org/matrix-spec-proposals/pull/3401) | Parent MSC — MSC3898 was split from it |
| [MSC4195](https://github.com/matrix-org/matrix-spec-proposals/pull/4195) | Successor approach: multi-SFU via federation, no cascading needed |
| [MSC4143](https://github.com/matrix-org/matrix-spec-proposals/pull/4143) | The framework MSC3898's problems are solved within |

---

## References

- PR: https://github.com/matrix-org/matrix-spec-proposals/pull/3898
- Rendered proposal: https://github.com/matrix-org/matrix-spec-proposals/blob/SimonBrandner/msc/sfu/proposals/3898-sfu.md
