# Support

## Documentation

| Surface                 | Where                                                   | Status                                                                       |
| ----------------------- | ------------------------------------------------------- | ---------------------------------------------------------------------------- |
| Framework docs          | <https://docs.mzizi.dev>                                | `mzizi-docs` holds only a README and LICENSE today — the site is being built |
| Component registry      | <https://mzizi.dev>                                     | `mzizi-registry` is the live registry and portal                             |
| Console                 | <https://app.mzizi.dev>                                 | `mzizi-console`                                                              |
| API                     | <https://api.mzizi.dev>                                 | `mzizi-api-gateway`                                                          |
| Language, charter, RFCs | [`mzizi-dev/mzizi`](https://github.com/mzizi-dev/mzizi) | `CHARTER.md` is the authoritative statement of what this project is for      |

## Where to raise what

| What                                                     | Where                                                                                    |
| -------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| A compiler bug, a language question, an `mz` CLI problem | An issue on [`mzizi`](https://github.com/mzizi-dev/mzizi/issues)                         |
| A component, token or registry-API problem               | An issue on [`mzizi-registry`](https://github.com/mzizi-dev/mzizi-registry/issues)       |
| Something wrong at app.mzizi.dev                         | An issue on [`mzizi-console`](https://github.com/mzizi-dev/mzizi-console/issues)         |
| Something wrong at api.mzizi.dev                         | An issue on [`mzizi-api-gateway`](https://github.com/mzizi-dev/mzizi-api-gateway/issues) |
| A documentation error                                    | An issue on [`mzizi-docs`](https://github.com/mzizi-dev/mzizi-docs/issues)               |
| Roadmap and sequencing                                   | [`mzizi-roadmap`](https://github.com/mzizi-dev/mzizi-roadmap/issues)                     |
| CI, governance, org standards, this file                 | An issue on [`mzizi-dev/.github`](https://github.com/mzizi-dev/.github/issues)           |
| A security vulnerability                                 | **Not an issue.** See [SECURITY.md](./SECURITY.md)                                       |

`agent-tools` is private. If your question is about the MCP server, `fundi`,
the CLI or the skills and you cannot open an issue there, raise it on
`mzizi-dev/.github` and it will be routed.

## What helps

- **For a compiler report**: the `.mz` source, the exact `mz` invocation, the
  full diagnostic, and the commit SHA. Not "main" — main moves.
- **For a WASM build failure**: say whether it reproduces on the host. A
  failure that only appears under `--target wasm32-unknown-unknown` is
  usually a different problem from one that appears natively, and that
  distinction is often the whole answer.
- **For a registry problem**: the component name and the install command you
  ran.

## Response times

Mzizi is a Bundu Foundation research project maintained by a small team. The
org has two members. Issues are read; a same-day reply is not promised.
Security reports are acknowledged within three working days.
