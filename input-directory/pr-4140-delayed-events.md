# PR #4140: MSC4140 Cancellable Delayed Events

**Source:** https://github.com/matrix-org/matrix-spec-proposals/pull/4140

## Metadata

- **State:** Open
- **Author:** @toger5 (Timo K, Element)
- **Created:** May 7, 2024
- **Last updated:** March 13, 2026
- **Branch:** `toger5/expiring-events-keep-alive` → `main`
- **Commits:** 140
- **Additions:** +859 lines
- **Labels:** voip, proposal, client-server, kind:feature, matrix-2.0, implementation-needs-checking
- **Rendered:** https://github.com/matrix-org/matrix-spec-proposals/blob/toger5/expiring-events-keep-alive/proposals/4140-delayed-events-futures.md

## PR Description (Implementations section)

```
Implementations:
- [x] Synapse
  - https://github.com/element-hq/synapse/pull/17326
- [x] Element Call running in SPA mode
  - https://github.com/matrix-org/matrix-js-sdk/pull/4294
  - https://github.com/element-hq/element-call/pull/2529
- [x] Element Web running Element Call in embedded mode
  - https://github.com/matrix-org/matrix-js-sdk/pull/4294
  - https://github.com/matrix-org/matrix-widget-api/pull/90
  - https://github.com/element-hq/element-call/pull/2529
- [x] Element X running Element Call in embedded mode
  - https://github.com/ruma/ruma/pull/1845
  - https://github.com/matrix-org/matrix-rust-sdk/pull/3600
  - https://github.com/element-hq/element-call/pull/2529

Known implementation gaps:
- [ ] 3ef314f861f3e54a9a474e624eba3684ee6ea978
- [ ] 95045cf00eeb2af0976b31d2262875ef7eefaeec
- [ ] 49b200dcc11de286974925177b1e184cd905e6fa
```

SCT stuff:
- [checklist](https://github.com/matrix-org/matrix-spec-proposals/pull/4140#issuecomment-2806245590)
- FCP not yet started
- Prior block lifted (~~Blocked~~)

## API Endpoints Added

- `PUT /_matrix/client/v3/rooms/{roomId}/delayed_event/{eventType}/{txnId}` — Schedule event
- `GET /_matrix/client/v3/delayed_events` — List delayed events
- `POST /_matrix/client/v3/delayed_events/{delay_id}` — Act on delayed event (`restart`, `send`, `cancel`)

## Companion

- MSC4309: https://github.com/matrix-org/matrix-spec-proposals/pull/4309 (sync notification when event fires)
