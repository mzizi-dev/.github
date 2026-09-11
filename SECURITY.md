# Security policy

This file is the org-wide fallback for `mzizi-dev`. GitHub shows it for any
repo in the org that does not ship its own `SECURITY.md`. Today that is every
repo except `mzizi-registry`, which has its own — and that one currently
points at a stale URL (see "Known problems" below).

## Reporting a vulnerability

**Do not open a public issue, pull request or discussion.**

Report privately through GitHub Security Advisories, on the repo the finding
affects:

| Repo                                                   | Private reporting  | Advisory link                                                         |
| ------------------------------------------------------ | ------------------ | --------------------------------------------------------------------- |
| `mzizi`                                                | **enabled**        | <https://github.com/mzizi-dev/mzizi/security/advisories/new>          |
| `mzizi-registry`                                       | **enabled**        | <https://github.com/mzizi-dev/mzizi-registry/security/advisories/new> |
| `mzizi-console`                                        | not enabled        | —                                                                     |
| `mzizi-api-gateway`                                    | not enabled        | —                                                                     |
| `mzizi-site`, `mzizi-docs`, `mzizi-roadmap`, `.github` | not enabled        | —                                                                     |
| `agent-tools`                                          | n/a — private repo | —                                                                     |

Verified 2026-09-11 against the GitHub API. If the repo you need is in the
"not enabled" rows, **report against `mzizi-dev/mzizi`** and say in the
report which component it actually concerns; it reaches the same maintainers.
Enabling private reporting on the remaining repos is a tracked gap — see
[ORG_STANDARDS.md](./ORG_STANDARDS.md#known-gaps).

If GitHub advisories are unavailable to you, email **<security@nyuchi.com>**
with the same information. Nyuchi Africa operates the Mzizi surfaces
commercially and that mailbox is the one already in use for this ecosystem;
it is not a separate team.

Expect an acknowledgement within **three working days**. A partial report
sent early is more useful than a complete one sent late.

## What to include

- The affected repo, and the surface — a URL, an endpoint, a crate, an MCP
  tool, a registry item.
- A reproduction. For `mzizi-api-gateway`, the request. For `mzizi`, the
  `.mz` source or the compiler invocation. For `mzizi-registry`, the item or
  API call.
- The version or commit SHA you were on.
- The impact you believe it has, and who is exposed.
- A proposed fix if you have one — patches are welcome through the advisory
  UI.

## Scope

In scope:

| Repo                       | Surface                                                                                                                                                         |
| -------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `mzizi`                    | The compiler and the `mz` CLI — anything where checking or compiling untrusted `.mz` source can execute code, escape the working directory, or exhaust the host |
| `mzizi-api-gateway`        | api.mzizi.dev — the Worker: routing, auth token handling, request and response validation                                                                       |
| `mzizi-registry`           | mzizi.dev — the registry API and the component source it serves                                                                                                 |
| `mzizi-console`            | app.mzizi.dev — the console and its WASM islands                                                                                                                |
| `mzizi-site`, `mzizi-docs` | The public site and documentation                                                                                                                               |
| This repo                  | The reusable workflows, and the ruleset definitions under `github-rulesets/`                                                                                    |

Out of scope: volumetric denial of service, findings against Cloudflare or
Vercel themselves, missing headers with no demonstrated impact, and raw
scanner output with no exploit path.

## What we treat as most serious

**Supply chain, above everything else.** `mzizi-registry` serves source code
and AI-facing instructions into downstream production apps across the Bundu
ecosystem, and `mzizi` compiles source that agents author automatically. A
finding that lets an attacker change what a downstream consumer installs, or
what a compiler emits, is critical regardless of how hard it is to reach —
because the blast radius is every consumer, not one endpoint.

Specifically:

1. **Anything that changes what the registry serves** for an existing
   component, or that causes a component to install something other than what
   its source says.
2. **Anything that lets untrusted `.mz` input influence the host** during a
   `mz check` or a compile — the whole design assumes an agent runs that in a
   loop, unattended, thousands of times.
3. **Credential exposure in the Worker** — `mzizi-api-gateway` holds the
   tokens for api.mzizi.dev.

## Disclosure

We confirm the report, agree a timeline with you, and credit you in the
advisory unless you ask otherwise. Please give a reasonable window to ship a
fix before publishing.

## Known problems with this policy

Stated here rather than quietly fixed elsewhere, because they affect where a
report actually lands:

- **`mzizi-registry`'s own `SECURITY.md` points at
  `https://github.com/nyuchi/mzizi/security/advisories/new`.** That repo was
  renamed to `mzizi-dev/mzizi-registry`; GitHub redirects the path, so the
  link happens to work today, but it names an org that no longer owns the
  code. It also cites version lines (4.0.x / 4.1.x) that pre-date the move.
  Fixing it is a PR against that repo.
- **Secret scanning and push protection are enabled on only two of the eight
  public repos** — `mzizi-registry` and `mzizi-api-gateway`. The other six,
  including `mzizi` itself, have both switched off. Push protection is what
  stops a secret ever reaching the remote; the `gitleaks` job in CI only
  catches what is already committed. They are complements, not alternatives,
  and `mzizi` and `mzizi-console` currently rely on the CI half alone.
  Dependabot security updates are off on every repo in the org.
