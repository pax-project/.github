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

Cross-cutting docs that don't belong to a single repo live here:

- [`docs/ideas/`](https://github.com/pax-project/.github/blob/main/docs/ideas/README.md) —
  feature idea backlog, split by which repo each idea affects.
- [`docs/goals/`](https://github.com/pax-project/.github/blob/main/docs/goals/README.md) —
  every repo's spec, implementation-status, and build history.
