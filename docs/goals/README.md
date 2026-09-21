# pax — goals & status

Every spec, implementation-status checklist, and build history for
`pax-core` and `lazy-pax` lives here rather than in either code repo — so a
repo's own files only ever describe what's true *now* (README, CLAUDE.md),
never a moving target of past/future goals that drifts out of sync with the
code sitting right next to it.

## pax-core

- [`pax-core/mvp.md`](pax-core/mvp.md) — the original, pre-implementation
  MVP spec. Historical: kept as written, not updated as things ship.
- [`pax-core/status.md`](pax-core/status.md) — current implementation
  status against that spec. Kept up to date; this is the one to check for
  "is X built yet."

## lazy-pax

- [`lazy-pax/dod.md`](lazy-pax/dod.md) — the original Definition of Done for
  the `lazypax` MVP. Historical, same role as `pax-core/mvp.md`.
- [`lazy-pax/status.md`](lazy-pax/status.md) — current implementation status
  against that DoD. Kept up to date.
- [`lazy-pax/build-log.md`](lazy-pax/build-log.md) — step-by-step history of
  how the MVP was built (bugs found, verification detail per step). Purely
  historical, not a status checklist.

See [`../ideas/`](../ideas/README.md) for what's *not* built yet and being
considered — this directory is about what was targeted and what's done
against it, not the feature backlog.
