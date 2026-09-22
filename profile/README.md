# pax-project

Org home for the **pax** research-paper toolkit: a Rust-based, Nix-backed
system for discovering, declaring, managing, and reproducibly acquiring
academic papers.

## Repos

- [pax-core](https://github.com/pax-project/pax-core) — the library and `pax` CLI:
  provider search, `papers.nix` management, Nix-backed artifact fetching.
- [lazy-pax](https://github.com/pax-project/lazy-pax) — `lazypax`, the terminal UI
  built on `pax-core`.
- `pax-ai` (planned) — shared AI substrate (summaries, semantic search,
  RAG) and an MCP server, depending on `pax-core` without modifying it.

## Docs

Idea backlogs and implementation status live in each repo's Issues (or
Discussions, for `pax-ai`) and the org's shared Project (see the
Projects tab above).
[`docs/`](https://github.com/pax-project/.github/blob/main/docs/README.md)
holds only fixed architectural boundaries that don't fit those tools.
