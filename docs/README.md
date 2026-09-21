# pax — docs

Docs for the whole `pax` project, organized by which repo they're about —
not by `pax-core` or `lazy-pax` themselves, so neither repo's own docs
describe a moving target (past/future goals, a feature backlog) alongside
its actual code.

## By repo

- [`pax-core/`](pax-core/README.md) — spec, status, and idea backlog for
  [`pax-core`](https://github.com/pax-project/pax-core).
- [`lazy-pax/`](lazy-pax/README.md) — spec, status, build history, and idea
  backlog for [`lazy-pax`](https://github.com/pax-project/lazy-pax).
- [`pax-ai/`](pax-ai/README.md) — idea backlog for the planned `pax-ai`
  crate (not a repo yet).

Each repo's folder holds two different kinds of doc:

- **Goals** (`mvp.md`/`dod.md`, `status.md`, `build-log.md`) — what was
  targeted, and what's actually built against it today. Tracked against
  real code, kept up to date (except the spec/build-log themselves, which
  are deliberately historical).
- **Ideas** (`ideas.md`) — features nobody's built yet, not tracked against
  anything. Status values: `idea` (not started) · `prototyping` ·
  `shipped` · `rejected`. Review periodically and update status + notes as
  thinking changes.

Most ideas started as AI-assisted feature ideas; the rest were surfaced
from `pax-core`'s and `lazy-pax`'s original MVP non-goals lists — deferred
features that don't belong in a "non-goal" list once there's a backlog to
track them in. Only genuine architectural boundaries (division of
responsibility with Nix, no shadow state, etc.) stay documented as
non-goals in each repo's own `mvp.md`/`dod.md`.

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
