# Tasks: Fix Electron-only React update loop on new chat

## Goal

Creating a new chat in the Electron desktop app must not trigger React error #185 / maximum update depth. Browser web UI currently does not reproduce, so the fix should account for desktop's faster bootstrap/config/welcome timing.

## Scope

- apps/web route/state handling for new-chat drafts and draft-to-server promotion.
- Desktop-amplified React effect/update loops caused by unstable object dependencies or repeated navigation/state writes.
- Avoid broad ChatView refactors unless the narrower route/draft stabilization is insufficient.

## Recon findings

- Desktop immediately authenticates, starts server state sync, receives config/welcome/shell updates, and uses hash history; this can amplify update loops that web timing hides.
- Server thread route still has effects depending on `threadRef` object identity after previous fix.
- Draft route still constructs a fresh canonical object from `serverThread`; its effects use primitive ids, but render branch still uses the object.
- ChatView remains a secondary suspect due to many effects, but the highest-confidence remaining issue is route/effect dependencies and repeated navigation/finalization.

## Implementation slices

1. Remove `threadRef` object identity from server-route effect dependency arrays by building refs inside effects/callbacks from primitive route params where needed.
2. Make draft route canonicalization primitive-first; only construct objects for rendering/call boundaries after primitive canonical ids are known.
3. Add idempotence guards around replace navigation/finalization if repeated identical navigation can occur under Electron hash history.
4. Run `bun fmt`, `bun lint`, and `bun typecheck`.
