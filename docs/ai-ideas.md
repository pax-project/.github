# lazypax — AI capability ideas

A running list of AI-assisted features to consider post-MVP, grouped by which
part of the app they'd improve. `docs/dod.md` explicitly excluded AI
summaries, full-text indexing, citation graphs, and related-paper
recommendations from the MVP — this is the backlog for revisiting them now
that the MVP loop is closed.

Status values: `idea` (not started) · `prototyping` · `shipped` · `rejected`.
Review periodically and update status + notes as thinking changes.

---

## Architecture: `pax-ai`

None of the ideas below live in `pax-core`. `pax-core`'s value is being a
small, deterministic, reproducible library (`git clone` + `nix build`
reconstructs the same artifact) — adding LLM calls, embeddings, or a derived
cache to it would dilute that contract even if gated behind a feature flag:
it stops being obviously reproducible, gains a second state model alongside
`papers.nix`, and couples unrelated version bumps together.

Instead, AI-heavy ideas (#1, #2, #3, #4, #5, #7, #8, #9) are planned as a new
sibling crate — working name `pax-ai` — that depends on `pax-core` the same
way `lazy-pax` does (a client, not a fork, no modifications to `pax-core`
itself). `lazy-pax` will depend on `pax-ai` for these features instead of
implementing them internally, and `pax-ai` is also where the MCP server
(#9) lives. This gives every AI feature one shared implementation instead of
duplicating it between the TUI and any MCP client.

Ideas that don't need an LLM/embedding step (currently just #6) can stay as
plain `lazy-pax` logic — there's no reason to route a pure heuristic through
`pax-ai`.

`pax-ai`'s own non-negotiables, carried over from `pax-core`'s: writes only
ever go through `pax-core` functions (never hand-write `papers.nix`), and any
persisted state (embeddings, cached summaries) is a rebuildable cache derived
purely from `papers.nix` via `pax-core`'s read functions — never a second
source of truth. Crate name isn't finalized.

---

## Search / finding quality

Improve the moment between typing a query and deciding what's worth adding.

### 1. AI paper summaries
- **Status:** idea
- **What:** On add/fetch, generate a short summary from the abstract (or
  extracted PDF text once fetched) shown in the detail view.
- **Why here:** Helps decide whether a search result is worth adding, and
  gives a faster refresher on already-declared papers.
- **Notes:** Lives in `pax-ai`; `lazy-pax` calls into it rather than
  implementing summarization itself. No `pax-core` API change needed — reads
  data already fetched via `pax-core`. Needs an LLM API call, so needs a way
  to run it off the UI thread (same `job.rs` pattern as search/fetch).

### 4. Related-paper / "you might also want" suggestions
- **Status:** idea
- **What:** Embedding similarity across search results and/or your existing
  library, surfaced in the detail view ("similar to 3 papers you've already
  added").
- **Why here:** Surfaces papers you wouldn't have found by keyword search
  alone, while still deciding what to add.
- **Notes:** Shares `pax-ai`'s embedding infrastructure with #2. Could start
  library-only (compare a new candidate against declared papers) before
  extending to cross-provider result clustering.

---

## Local library exploration

Improve how you navigate/query papers you've already declared.

### 2. Semantic / natural-language search over your library
- **What:** Embed declared papers' abstracts, let you query "papers about X"
  instead of only exact keyword/author/tag filters
  (`pax_core::filter_papers`).
- **Status:** idea
- **Why here:** The current library filter is exact-match on
  author/year/tag; this adds fuzzy, meaning-based recall.
- **Notes:** The embedding cache lives in `pax-ai`, rebuilt from
  `papers.nix` via `pax-core`'s read functions — not a second source of
  truth, and never written by `pax-ai` directly. First feature that needs
  `pax-ai`'s cache design settled, since #4/#5 build on it.

### 5. RAG chat over your library
- **Status:** idea
- **What:** Ask questions across all fetched PDFs ("which of my papers
  discuss X"), answered with citations back to specific papers.
- **Why here:** Turns the library from a list you scroll into something you
  can query directly.
- **Notes:** Builds on #2's embedding infrastructure in `pax-ai`. Needs
  full-text extraction from fetched PDFs — bigger lift than the others on
  this list.

---

## Metadata quality

Improve the accuracy/completeness of what's recorded about each paper.

### 3. Auto-tagging
- **Status:** idea
- **What:** Suggest tags/notes based on abstract content when a paper is
  added; user confirms before anything is saved.
- **Why here:** Reduces manual tagging effort without giving the AI direct
  write access — suggestion, not autonomous edit, keeps `edit_paper` as the
  sole write path.
- **Notes:** Suggestion logic lives in `pax-ai`; the actual write still goes
  through `lazy-pax` calling `pax_core::edit_paper` after user confirmation,
  same as today — `pax-ai` never writes to the library itself.

### 6. Duplicate / near-duplicate detection
- **Status:** idea
- **What:** Flag likely duplicates beyond the current DOI-exact match (e.g.
  same paper indexed differently across providers, preprint vs.
  published version).
- **Why here:** Current in-library indicator relies on `normalize_doi`
  matching; this catches the cases that slip through.
- **Notes:** Can likely be a plain heuristic (title/author fuzzy matching) —
  no LLM/embedding step required, so this one can stay `lazy-pax`-only
  rather than routing through `pax-ai`. Revisit if it later needs semantic
  matching instead of string similarity.

### 7. Citation-key / BibTeX quality pass
- **Status:** idea
- **What:** AI-assisted key naming or BibTeX field completion when a
  provider record is sparse (missing venue, incomplete author list, etc.).
- **Why here:** Improves export quality (`pax_core::bibtex::render`) without
  touching how records are stored.
- **Notes:** Lives in `pax-ai`; suggestion only, same confirm-before-write
  pattern as #3.

---

## Reading quality

Improve the experience once a paper is actually open.

### 8. Reading-companion mode
- **Status:** idea
- **What:** While a paper is open in the external PDF viewer, show a side
  panel extracting key claims / methodology / limitations from the PDF
  text.
- **Why here:** The DoD explicitly rules out in-terminal PDF
  rendering/annotation — this is a complementary side panel, not a PDF
  viewer replacement, so it doesn't reopen that non-goal.
- **Notes:** Needs full-text extraction (shared need with #5) — lives in
  `pax-ai` alongside that infrastructure. Depends on the viewer/lazypax
  being able to run concurrently, which they already do (`open` just
  launches an external process).

---

## Agent / LLM access

A different axis from the sections above: instead of AI features embedded
*inside* the TUI, let an external agent/LLM drive the library directly.

### 9. MCP server over `pax-core`, hosted in `pax-ai`
- **Status:** idea
- **What:** A new binary exposing `search_all`, `add_candidate`,
  `fetch_paper`, `filter_papers`/`list`, `edit_paper`, `remove_paper`,
  `bibtex::render`, etc. as MCP tools, so any MCP client (Claude Code, Claude
  Desktop, other agents) can drive the library with natural-language
  requests ("find and add papers about X"). Since it lives in `pax-ai`, it
  can also expose #1/#2/#5's AI features as tools (e.g. "summarize this
  paper", "semantic search my library") alongside the plain CRUD ones.
- **Why here:** `pax-ai` already depends on `pax-core` as a peer client, the
  same way `lazy-pax` does — the MCP server is just another consumer of that
  same dependency, bound by the same non-negotiables (no shelling out to
  `pax`, no shadow state, writes only through `pax-core` functions). Hosting
  it in `pax-ai` rather than `pax-core` keeps `pax-core`'s dependency
  footprint and reproducibility contract untouched.
- **Notes:** Destructive tools (`remove_paper`) should require explicit
  confirmation from the calling agent/user, mirroring `lazy-pax`'s own
  confirm-prompt for remove. Recommended starting point among the ideas on
  this list — doesn't strictly require any AI feature above to exist first,
  since the plain CRUD tools have no LLM dependency of their own.

---

## Cross-cutting considerations for all of the above

- **Architecture fit:** AI-heavy ideas (#1, #2, #3, #4, #5, #7, #8, #9) are
  `pax-ai` functionality, consumed by `lazy-pax`, not code written directly
  into either `lazy-pax` or `pax-core`. #6 can stay `lazy-pax`-only as a
  plain heuristic. None of these require writing new fields into
  `papers.nix` (which would need a `pax-core` change first, and per the DoD,
  that's raised upstream, not patched around locally) — see the
  "Architecture: `pax-ai`" section above for the full reasoning.
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
  decision now belongs to `pax-ai`, not `pax-core` or `lazy-pax`.
