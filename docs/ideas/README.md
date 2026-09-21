# pax — idea backlog

AI-assisted feature ideas for the pax project, split by which repo they'd
land in. `lazy-pax`'s `docs/dod.md` explicitly excluded AI summaries,
full-text indexing, citation graphs, and related-paper recommendations from
the `lazypax` MVP — this backlog is for revisiting them now that the MVP
loop is closed.

Status values: `idea` (not started) · `prototyping` · `shipped` · `rejected`.
Review periodically and update status + notes as thinking changes.

## By repo

- [pax-core.md](pax-core.md) — currently empty; see "Architecture" below for why.
- [lazy-pax.md](lazy-pax.md) — 1 idea.
- [pax-ai.md](pax-ai.md) — 8 ideas; where most AI-assisted feature work lands.

---

## Architecture: `pax-ai`

None of the ideas here live in `pax-core`. `pax-core`'s value is being a
small, deterministic, reproducible library (`git clone` + `nix build`
reconstructs the same artifact) — adding LLM calls, embeddings, or a derived
cache to it would dilute that contract even if gated behind a feature flag:
it stops being obviously reproducible, gains a second state model alongside
`papers.nix`, and couples unrelated version bumps together.

Instead, AI-heavy ideas are planned as a new sibling crate/repo — working
name `pax-ai` — that depends on `pax-core` the same way `lazy-pax` does (a
client, not a fork, no modifications to `pax-core` itself). `lazy-pax` will
depend on `pax-ai` for these features instead of implementing them
internally, and `pax-ai` is also where the MCP server lives. This gives
every AI feature one shared implementation instead of duplicating it
between the TUI and any MCP client.

Ideas that don't need an LLM/embedding step can stay as plain `lazy-pax`
logic — there's no reason to route a pure heuristic through `pax-ai`.

`pax-ai`'s own non-negotiables, carried over from `pax-core`'s: writes only
ever go through `pax-core` functions (never hand-write `papers.nix`), and any
persisted state (embeddings, cached summaries) is a rebuildable cache derived
purely from `papers.nix` via `pax-core`'s read functions — never a second
source of truth. Crate/repo name isn't finalized.

## Cross-cutting considerations for all of the above

- **Architecture fit:** AI-heavy ideas are `pax-ai` functionality, consumed
  by `lazy-pax`, not code written directly into either `lazy-pax` or
  `pax-core`. None of these require writing new fields into `papers.nix`
  (which would need a `pax-core` change first, and per `lazy-pax`'s DoD,
  that's raised upstream, not patched around locally).
- **No shadow state:** anything that needs persistence (embeddings, cached
  summaries) is a rebuildable cache in `pax-ai`, derived purely from
  `papers.nix` via `pax-core`'s read functions — never a second source of
  truth, and `pax-ai` never writes `papers.nix` directly.
- **Off-thread execution:** any LLM/API call follows the same
  `job.rs`/`spawn_blocking` pattern as provider search and `nix` calls, so
  the UI never freezes on a network round-trip.
- **Cost/privacy:** most of these send abstracts or full paper text to a
  third-party API — worth a deliberate decision (which provider, local vs.
  hosted model, opt-in/opt-out) before picking a starting point. This
  decision belongs to `pax-ai`, not `pax-core` or `lazy-pax`.
