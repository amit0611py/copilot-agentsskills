# Angular 13 -> 21 upgrade runbook

Source of truth for versions: https://angular.dev/reference/versions
Source of truth for breaking changes per hop: https://angular.dev/update-guide

## Node / TypeScript matrix

Set Node **before** running anything in a hop. Values below are from the official Angular
compatibility table.

| Hop | Angular | Node.js required | TypeScript | Use this Node |
|-----|---------|------------------|------------|---------------|
| — | 13.x (start) | ^12.20 \|\| ^14.15 \|\| ^16.10 | >=4.4.3 <4.7 | 16.20.x |
| 1 | 14.x | ^14.15 \|\| ^16.10 | >=4.6.2 <4.9 | 16.20.x |
| 2 | 15.x | ^14.20 \|\| ^16.13 \|\| ^18.10 | >=4.8.2 <5.0 | 16.20.x |
| 3 | 16.x | ^16.14 \|\| ^18.10 | >=4.9.3 <5.2 | 18.19.x |
| 4 | 17.x | ^18.13 \|\| ^20.9 | >=5.2 <5.5 | 18.19.x |
| 5 | 18.x | ^18.19.1 \|\| ^20.11.1 \|\| ^22.0 | >=5.4 <5.6 | 18.19.1+ |
| 6 | 19.x | ^18.19.1 \|\| ^20.11.1 \|\| ^22.0 | >=5.5 <5.9 | 20.11.1+ |
| 7 | 20.x | ^20.19 \|\| ^22.12 \|\| ^24.0 | >=5.8 <6.0 | 22.12+ |
| 8 | 21.x | ^20.19 \|\| ^22.12 \|\| ^24.0 | >=5.9 <6.0 | 22.12+ |

RxJS stays at `^7.4.0` across every hop. If the repo is still on RxJS 6, migrate to 7 in
its own PR before hop 1.

Three Node "eras": 16.20 covers hops 1–2, 18.19 covers hops 3–5, 22.12 covers hops 6–8.
Pin the era in `.nvmrc` per branch and in CI.

## Risk ranking

| Hop | Risk | Why |
|-----|------|-----|
| 14 -> 15 | **Highest** | Angular Material MDC rewrite. DOM and CSS class names change. Take the `legacy-*` migration to stay behaviour-neutral. |
| 16 -> 17 | High | Node 16 dropped, new esbuild/Vite application builder, new control flow, Material legacy components removed — MDC adoption must be a finished, separate PR before this hop. |
| 15 -> 16 | Medium | `ngCspNonce` / `CSP_NONCE` become available. Unblocks backend `style-src` work. |
| 20 -> 21 | Medium | Test runner and template-debug-attribute changes; verify against the official update guide before starting, not from memory. |
| others | Low–Medium | Mostly schematic-handled. |

## Visual baseline (do this first, before hop 1)

Design cannot be "not broken" if nothing measures it. Create
`docs/upgrade/visual-baseline/` with full-page Playwright screenshots at 3 viewports
(360, 768, 1440) for at least:

- login / auth
- dashboard or landing after auth
- every payment entry screen
- payment confirmation and receipt
- every modal and every form validation error state
- any print/PDF view
- one screen per third-party widget (date picker, charts, tables)

Each hop PR re-runs these and attaches the diff.

## Payment smoke flows (manual, every hop)

1. New card payment, success.
2. New card payment, declined.
3. 3DS / redirect-based challenge, including the return leg.
4. Saved instrument payment.
5. Refund or void, if in scope.
6. Session timeout mid-payment.

## Rollback

Each hop lives on its own branch off `main`, merged only after DoD passes. Rollback = revert
the single merge commit. Do not stack hops on one another before merge; if hop 3 is bad you
must be able to sit on hop 2 in production indefinitely.

## Recorded metrics

| Version | Prod bundle (initial) | Prod bundle (total) | Build time | Test time |
|---------|----------------------|---------------------|-----------|-----------|
| 13 (baseline) | | | | |

Fill this in per hop.
