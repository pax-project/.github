# pax-core — ideas

Ideas that live directly in `pax-core` — mostly "another client of the
library" or "another provider," matching how `pax`, `lazypax`, and the
planned `pax-ai`/MCP server are all just clients of the same library. None
of these need an LLM call, embeddings, or a derived cache; see
[pax-ai.md](pax-ai.md) for the ones that do.

Surfaced from `pax-core`'s original MVP non-goals list (`docs/mvp.md §5`) —
each was a deferred feature, not an architectural boundary, so it's tracked
here rather than left as a stale "not doing this" note.

---

### 11. Plugin / extension system for pax-core
- **Status:** idea
- **What:** A way for third-party code to extend `pax-core`'s behavior
  (custom providers, custom export formats, etc.) without forking the
  library.
- **Why here:** Every current "extension point" (a new `Provider` impl, a
  new export format) requires a PR against `pax-core` itself; a plugin
  system would let that happen out-of-tree.
- **Notes:** No concrete design yet — worth scoping only if a real
  third-party extension need shows up, rather than speculatively.

### 12. Web interface
- **Status:** idea
- **What:** A web UI client of `pax-core`, alongside `pax` (CLI), `lazypax`
  (TUI), and the planned MCP server.
- **Why here:** Same "just another client" shape as `lazypax` — no new
  `pax-core` domain logic needed, just a new adapter.
- **Notes:** Would need a decision on how it talks to `pax-core` (a local
  web server embedding the library directly vs. going through the planned
  MCP server) before implementation starts.

### 13. Citation graph
- **What:** Fetch and expose citation relationships between papers (who
  cites whom), not just similarity — distinct from
  [pax-ai.md #4](pax-ai.md)'s embedding-based "related papers," which infers
  similarity from abstracts rather than reading real citation edges.
- **Status:** idea
- **Why here:** Several providers (Semantic Scholar, OpenAlex) already
  expose citation/reference lists per paper — this is provider data
  `pax-core` doesn't fetch yet, not an AI feature.
- **Notes:** `lazypax` (or a future web interface) would be the natural
  place to visualize the resulting graph; this entry is just the
  `pax-core`-side data-fetching capability.

### 14. Neovim integration
- **Status:** idea
- **What:** A Neovim plugin/interface driving `pax-core`, for users who want
  their research library reachable from their editor.
- **Why here:** Another `pax-core` client, same shape as `lazypax`.
- **Notes:** Lowest-priority of the "new client" ideas here — no signal yet
  that this is more valuable than the web interface or MCP server.

### 15. Zotero synchronization
- **Status:** idea
- **What:** Import from / export to Zotero, so a `pax` library can
  interoperate with an existing Zotero collection.
- **Why here:** Metadata interchange, similar in spirit to
  `pax_core::bibtex::render` but bidirectional and Zotero-specific.
- **Notes:** Needs a decision on conflict handling (a paper edited in both
  places) before implementation — not just a one-way export.

### 16. Cloud synchronization
- **Status:** idea
- **What:** Sync a `research/` library's state across machines without a
  shared git remote.
- **Why here:** `pax` is currently local-first by design (`papers.nix` +
  git is the only sync story); this would be a deliberate expansion of that
  philosophy, not a small addition.
- **Notes:** Worth a real design discussion before starting — this is the
  idea on this list most likely to conflict with an existing non-negotiable
  (git-as-sync is currently load-bearing for the reproducibility story).
