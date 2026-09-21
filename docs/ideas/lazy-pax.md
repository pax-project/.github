# lazy-pax — ideas

Ideas that live directly in `lazy-pax` — either ones that don't need an
LLM/embedding step (see [README.md](README.md) for the `pax-ai` split), or
ones that are inherently UI/client concerns rather than `pax-core` domain
logic.

Ideas 17–20 are surfaced from `lazy-pax`'s original MVP non-goals list
(`docs/dod.md §4`) — each was a deferred feature, not an architectural
boundary, so they're tracked here rather than left as stale "not doing
this" notes.

---

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

### 17. In-terminal PDF preview, rendering, and annotation
- **Status:** idea
- **What:** Render and/or annotate a paper's PDF inside the TUI, instead of
  always shelling out to an external viewer.
- **Why here:** The MVP deliberately launches `$PAX_PDF_VIEWER` externally;
  this would be a genuinely large addition (a terminal PDF renderer) rather
  than a small one. Distinct from
  [pax-ai.md #8](pax-ai.md)'s reading-companion side panel, which
  complements the external viewer instead of replacing it.
- **Notes:** Biggest lift on this list — no existing Rust terminal PDF
  renderer is a drop-in choice; would need real spike/research first.

### 18. Scripting / plugin system for lazypax
- **Status:** idea
- **What:** Let users script `lazypax` actions or extend it with plugins,
  beyond the current fixed vim-style keybindings.
- **Why here:** The MVP's keybinding scheme is fixed and non-extensible by
  design; this would open it up.
- **Notes:** No concrete design yet — worth scoping only against a real
  workflow a fixed keymap can't cover, rather than speculatively.

### 19. Config/theming system
- **Status:** idea
- **What:** Configurable colors/theme, beyond the current
  `PAX_PDF_VIEWER` + library-path configuration.
- **Why here:** The MVP has no theming at all — this is a straightforward,
  self-contained UI feature.
- **Notes:** Lowest-effort idea on this list; a reasonable first pick if
  someone wants a small, well-scoped `lazypax` task.

### 20. Multi-library / workspace switching
- **Status:** idea
- **What:** Switch between multiple `research/` libraries within one
  running `lazypax` instance, instead of one library per launch (current
  behavior, tied to CWD).
- **Why here:** Useful for anyone maintaining more than one research
  library (e.g. separate personal/work libraries).
- **Notes:** Touches more of the app's state model than it might look
  (every screen currently assumes one library) — worth a design pass before
  implementation.
