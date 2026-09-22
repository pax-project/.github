# pax — docs

Docs for the whole `pax` project, organized by which repo they're about —
not by `pax-core` or `lazy-pax` themselves, so neither repo's own docs
describe a moving target (past/future goals, a feature backlog) alongside
its actual code.

## By repo

- [`lazy-pax/`](lazy-pax/README.md) — non-negotiable architectural
  constraints for [`lazy-pax`](https://github.com/pax-project/lazy-pax).

`pax-core` and `pax-ai` have no file-based docs left here — their idea
backlogs and implementation-status tracking moved to each repo's Issues
(or Discussions, for `pax-ai`, which isn't a repo yet) and a shared
org-wide Project, and `pax-core`'s original MVP spec now lives in that
repo's `1.0.0` Release notes.

This folder now only holds what those native tools can't express: fixed
architectural boundaries (division of responsibility with Nix, no shadow
state, etc.) that aren't backlog items or progress to track — see
`lazy-pax`'s `dod.md`.

---

## Architecture: `pax-ai`

Ideas that need an LLM call, embeddings, or a derived cache don't live in
`pax-core`. `pax-core`'s value is being a small, deterministic, reproducible
library (`git clone` + `nix build` reconstructs the same artifact) — adding
that kind of logic to it would dilute the contract even if gated behind a
feature flag: it stops being obviously reproducible, gains a second state
model alongside `papers.nix`, and couples unrelated version bumps together.

Instead, AI-heavy ideas are planned as a new sibling crate/repo — working
name `pax-ai` — that depends on `pax-core` the same way `lazy-pax` does (a
client, not a fork, no modifications to `pax-core` itself). `lazy-pax` will
depend on `pax-ai` for these features instead of implementing them
internally, and `pax-ai` is also where the MCP server lives. This gives
every AI feature one shared implementation instead of duplicating it
between the TUI and any MCP client.

Ideas that don't need an LLM/embedding step — most of `pax-core/ideas.md`
and `lazy-pax/ideas.md` — are ordinary features in whichever repo they
affect; there's no reason to route a plain heuristic or a new UI screen
through `pax-ai`.

`pax-ai`'s own non-negotiables, carried over from `pax-core`'s: writes only
ever go through `pax-core` functions (never hand-write `papers.nix`), and any
persisted state (embeddings, cached summaries) is a rebuildable cache derived
purely from `papers.nix` via `pax-core`'s read functions — never a second
source of truth. Crate/repo name isn't finalized.

## Cross-cutting considerations for AI ideas specifically

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
