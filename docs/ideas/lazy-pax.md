# lazy-pax — ideas

Ideas that live directly in `lazy-pax` — currently just the one that doesn't
need an LLM/embedding step, so it has no reason to route through `pax-ai`.
See [README.md](README.md) for the overall backlog and the `pax-ai` split.

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
