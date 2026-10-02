# Mzizi

**The open architecture and design system of the Bundu ecosystem.**

_Mzizi_ means _root_. It is a Bundu Foundation project, and every Mukoko
and Nyuchi surface is built on it.

[mzizi.dev](https://mzizi.dev) · [Docs](https://docs.mzizi.dev) ·
[Registry API](https://api.mzizi.dev/v1/ui) ·
[The Nyuchi Architecture, §14](https://github.com/nyuchi/.github/blob/main/profile/canonical/NYUCHI_ARCHITECTURE.md#14-design-system--mzizi)

---

## What lives here

| Repository                                                            | What it is                                                                                                                                                                                                             |
| --------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`mzizi-registry`](https://github.com/mzizi-dev/mzizi-registry)       | The component registry: the components, the brand system and token pipeline, and the DNA-helix frontend architecture. Installs like any shadcn registry from `api.mzizi.dev`. Apache 2.0.                              |
| [`mzizi`](https://github.com/mzizi-dev/mzizi)                         | The Mzizi language: a language, compiler and runtime designed for machine authorship and designed to lower to Rust. A research project at Phase 0 — its README says plainly what is built and what is not. Apache 2.0. |
| [`mzizi-docs`](https://github.com/mzizi-dev/mzizi-docs)               | The documentation at [docs.mzizi.dev](https://docs.mzizi.dev).                                                                                                                                                         |
| [`mzizi-site`](https://github.com/mzizi-dev/mzizi-site)               | [mzizi.dev](https://mzizi.dev), which brings the language, the registry and the docs together.                                                                                                                         |
| [`mzizi-api-gateway`](https://github.com/mzizi-dev/mzizi-api-gateway) | `api.mzizi.dev`, the registry API, as a Rust Cloudflare Worker.                                                                                                                                                        |
| [`mzizi-roadmap`](https://github.com/mzizi-dev/mzizi-roadmap)         | The Mzizi roadmap.                                                                                                                                                                                                     |
| [`packages-npm`](https://github.com/mzizi-dev/packages-npm)           | Shared, publishable UI packages: Nyuchi's implementation of the Mzizi architecture.                                                                                                                                    |

## The design system in brief

- **Seven Minerals.** Cobalt, Sodalite, Tanzanite and Malachite (deep
  earth); Gold, Copper and Terracotta (hand). Colour is contract:
  choosing a colour is choosing its role.
- **Brands by role.** Mukoko is Tanzanite, Nyuchi is Gold, Shamwari is
  Sodalite, Bundu is Copper.
- **The helix.** Nodes on an engineering backbone, rungs across both
  backbones. The node set is deliberately uncapped.
- **Fundi.** The self-healing agent turns assurance signals into
  labelled GitHub issues. It never auto-merges.

## The canonical documents

- [The Nyuchi Architecture](https://github.com/nyuchi/.github/blob/main/profile/canonical/NYUCHI_ARCHITECTURE.md),
  which wins wherever documents disagree
- [The Mukoko Manifesto](https://github.com/mukoko-dev/.github/blob/main/profile/canonical/MUKOKO_MANIFESTO.md)
- [The Bundu Order](https://github.com/bundu-labs/.github/blob/main/profile/canonical/BUNDU_ORDER.md),
  whose pattern language is part of Mzizi's genetic code

## Contributing

Engineering rules and the lint gate are shared across the estate from
[`nyuchi/.github`](https://github.com/nyuchi/.github). Read its
[`CONTRIBUTING.md`](https://github.com/nyuchi/.github/blob/main/CONTRIBUTING.md)
and [`AGENTS.md`](https://github.com/nyuchi/.github/blob/main/AGENTS.md),
and this organisation's
[`CONTRIBUTING.md`](https://github.com/mzizi-dev/.github/blob/main/CONTRIBUTING.md),
before opening a PR.

_A project of the [Bundu Foundation](https://www.bundu.org)_
