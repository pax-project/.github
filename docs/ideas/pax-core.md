# pax-core — ideas

No items in the idea backlog currently target `pax-core` directly.

Ideas that need an LLM call, embeddings, or a derived cache are deliberately
kept out of `pax-core` to preserve its reproducibility contract (`git clone`
+ `nix build` reconstructs the same artifact, deterministically, with no
third-party API surface). See [README.md](README.md#architecture-pax-ai) for
the full reasoning — that logic lives in `pax-ai` instead, which depends on
`pax-core` as a client without modifying it.
