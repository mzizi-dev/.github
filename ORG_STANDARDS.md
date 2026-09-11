# Org CI and governance standards — `mzizi-dev`

**Everything here was read off the GitHub API**, not inferred from what ought
to be true. Where something is a proposal rather than something running today,
it is in [Known gaps](#known-gaps) and says so. Nothing on this page should be
read as enforced unless it says it is.

If you are adding CI to a repo in this org, the two sections worth reading
first are [The merge convention](#the-merge-convention) and
[Rust CI: the WASM trap](#rust-ci-the-wasm-trap).

> **Correction, 2026-09-12**
>
> **This document said the org was merge-only. It is now rebase-only, and the
> reversal is total.** Re-read off the API on 2026-09-12, every one of the nine
> repos reports:
>
> ```text
> allow_merge_commit   false      (was: true everywhere)
> allow_squash_merge   false      (unchanged)
> allow_rebase_merge   true       (was: false everywhere)
> allow_auto_merge     true       (was: true on three of nine)
> ```
>
> The same is true across all 75 repositories in the Bundu Foundation
> enterprise, so this is an estate-wide change rather than something done to
> this org alone. `gh pr merge --merge` is now the command that gets rejected;
> `gh pr merge --rebase --auto` is the one that works.
>
> Three other sections went stale in the same twenty-four hours and are
> corrected in place below, each marked with the date: the org had no rulesets
> and now has two, three repos were described as empty or README-only and now
> carry content and CI, and the per-repo settings table has moved on.
>
> Everything not marked with a 2026-09-12 date is still as it was read on
> 2026-09-11 and has not been re-verified.

The estate-wide README standard that governs how every repo in all seven orgs
opens now lives in the `nyuchi` org — see
[Writing a README](#writing-a-readme) at the end of this document.

---

## The org

Nine repos. Two members — `@bryanfawcett` (admin) and `@michellellawson`
(member). **No teams exist**, which is why `.github/CODEOWNERS` names users
rather than a `@mzizi-dev/...` handle.

| Repo                                                                  | Public | Stack                                    | State                                                                                                                                                                         |
| --------------------------------------------------------------------- | ------ | ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`mzizi`](https://github.com/mzizi-dev/mzizi)                         | yes    | Rust                                     | The language, compiler and `mz` CLI. Bundu Foundation IP. Has content and CI                                                                                                  |
| [`mzizi-registry`](https://github.com/mzizi-dev/mzizi-registry)       | yes    | Next.js (+ Rust crates)                  | The component registry. The largest repo, and the only one with its own community-health files. **2026-09-12: it no longer serves mzizi.dev** — see the note under this table |
| [`mzizi-console`](https://github.com/mzizi-dev/mzizi-console)         | yes    | Astro + Rust/Dioxus WASM islands         | app.mzizi.dev. Has content and CI                                                                                                                                             |
| [`mzizi-api-gateway`](https://github.com/mzizi-dev/mzizi-api-gateway) | yes    | Pure-Rust Cloudflare Worker (workers-rs) | api.mzizi.dev. Has content and CI                                                                                                                                             |
| [`mzizi-site`](https://github.com/mzizi-dev/mzizi-site)               | yes    | Astro on Cloudflare Workers              | **2026-09-12: no longer empty, and it now serves the `mzizi.dev` apex.** Three pages, CI, and a custom domain attached outside version control                                |
| [`mzizi-docs`](https://github.com/mzizi-dev/mzizi-docs)               | yes    | Mintlify                                 | **2026-09-12: no longer README-only.** The full documentation site, with CI. Not deployed — `docs.mzizi.dev` does not resolve                                                 |
| [`mzizi-roadmap`](https://github.com/mzizi-dev/mzizi-roadmap)         | yes    | —                                        | A pointer to `mzizi/design/ROADMAP.md`, which is where the roadmap now lives. Its README calls itself archived; **the repo is not archived on GitHub**                        |
| [`agent-tools`](https://github.com/mzizi-dev/agent-tools)             | **no** | TypeScript / pnpm                        | MCP server, `fundi` agent, CLI, skills. The most CI of any repo — ten workflows                                                                                               |
| [`.github`](https://github.com/mzizi-dev/.github)                     | yes    | —                                        | This repo                                                                                                                                                                     |

**2026-09-12 — the `mzizi.dev` apex changed hands.** It is served by the Astro
Worker in `mzizi-site`, not by `mzizi-registry` on Vercel:
`curl -sSI https://mzizi.dev` returns `server: cloudflare` with no
`x-vercel-id`, and `https://mzizi.dev/llms.txt` is byte-identical to
`public/llms.txt` in `mzizi-site`. The registry's developer portal —
`/components`, `/tokens`, `/brand`, `/architecture`, `/observability`, `/r/` —
now 404s on the apex and has no live address. `api.mzizi.dev` and
`mcp.mzizi.dev` survived because they had already moved. The cutover runbook
(`mzizi-site#3`) and the PR porting the displaced pages (`mzizi-site#5`) are
both still open, and `mzizi-registry#334`, which would have made the route
reviewable, was closed unmerged. `mzizi-site/README.md` records the sequence.

---

## The merge convention

**Corrected 2026-09-12. This section previously said the opposite, and said it
with a verification note attached, which is why it is worth reading twice.**

**Rebase merging is the only method enabled on all nine repos. Merge commits
and squash merging are both disabled, and auto-merge is on everywhere.** Read
off the API per repo on 2026-09-12:

| Setting              | Value on all nine | Was, on 2026-09-11 |
| -------------------- | ----------------- | ------------------ |
| `allow_rebase_merge` | **true**          | false              |
| `allow_merge_commit` | **false**         | true               |
| `allow_squash_merge` | false             | false              |
| `allow_auto_merge`   | **true**          | true on three      |

This matches all 75 repositories in the Bundu Foundation enterprise, so it is
an estate-wide setting rather than a decision taken in this org.

Merge with:

```sh
gh pr merge <n> --rebase --auto
```

`--merge` and `--squash` are now what the repo settings reject. Never
`--admin`.

### What this displaces

`mzizi-dev/mzizi`'s `MIGRATION.md` §1.1 argued for merge-only on the grounds
that _"squash discards the per-commit reasoning this project depends on"_.
**Rebase preserves that reasoning**: every commit on the branch lands on `main`
individually, in order, with its message intact. What is lost is the merge
commit itself — the two-parent node and the `Merge pull request #N from …`
line that `git log --first-parent` used to summarise a branch with.

Three consequences:

**1. Your commits are still permanent, exactly as written.** Nothing folds
them together. Clean the branch up with an interactive rebase _on your own
branch_ before requesting review, and write messages that explain why. This
part did not change.

**2. `merge_commit_title` / `merge_commit_message` no longer do anything.**
Every repo still reports `MERGE_MESSAGE` / `PR_TITLE`, but with
`allow_merge_commit: false` no merge commit can be produced, so the PR title
never reaches `main` in any form. A PR-title lint is now purely a review-time
convention — it guards nothing in the history. Worth keeping, worth being
honest about what it does.

**3. `required_linear_history` is now consistent with the org**, where it used
to contradict it. Rebase merges produce linear history by construction. Gap 4
below is resolved by this change rather than by anyone fixing it.

**A note on the history in this repo.** `mzizi`, `mzizi-api-gateway` and
`agent-tools` carry two-parent `Merge pull request #N from mzizi-dev/...`
commits near the tip of `main`. Those are artefacts of the old setting. They
are not reproducible today and should not be read as the convention.

---

## What CI actually runs, per repo

Read from the workflow files in each repo's default branch.

### `mzizi` — 2 workflows

`ci.yml` triggers on push to `main` and pull requests to `main` **or**
`claude/**`, with `concurrency` cancelling superseded runs off `main`.

| Job (check name)   | What it runs                                                                                                                                                   |
| ------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `compiler`         | `cargo fmt -- --check`, `cargo clippy --all-targets -- -D warnings`, `cargo test`, all in `compiler/`                                                          |
| `compiler` (cont.) | `mz check ../examples/connectivity_bar.mz`, then `mz check` over **every** file in `primitives/`, using the built binary rather than the test harness          |
| `secret scan`      | `gitleaks detect` at `fetch-depth: 0` — full history, because this repo arrived via `git subtree split` and every commit reached CI for the first time at once |

`mzizi-lang-benchmark-dispatch.yml` is the second workflow. On push to
`main` it dispatches to a held-out benchmark runner named by the
`MZIZI_HELDOUT_REPO` repo variable, guarded by
`if: github.repository == 'mzizi-dev/mzizi'` so a fork cannot fire it. The
held-out task set is deliberately kept out of the public repo — per
`MIGRATION.md` §5, it is "withheld so the benchmark measures the language
rather than memorisation". Nothing here checks whether that runner exists.

The `mz check` steps are the interesting part and are worth copying in
spirit: they assert the _shipped binary_ still accepts the corpus, which is a
different claim from "the tests pass".

### `mzizi-console` — 1 workflow

| Job           | What it runs                                                                                                                                                                                                                                                                 |
| ------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `rust`        | fmt, clippy (host), `cargo test`, **and `cargo check --target wasm32-unknown-unknown --all-targets`**                                                                                                                                                                        |
| `web`         | pnpm 10.33.0, Node 22, `astro check`, `astro build`. The build is what proves the two toolchains compose — the island script references a bundle name derived from the crate name, so a rename that updates one and not the other fails here instead of serving a blank page |
| `secret scan` | gitleaks, full history                                                                                                                                                                                                                                                       |

### `mzizi-api-gateway` — 1 workflow

| Job            | What it runs                                                                                                                                                                                                                                                                                                                                                     |
| -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `rust`         | fmt; **clippy against `wasm32-unknown-unknown`**, not the host, because the `worker` crate's API is `cfg`'d for that target; `cargo check --target wasm32-unknown-unknown`; `cargo test`                                                                                                                                                                         |
| `worker build` | `cargo install worker-build --version ^0.8`, `worker-build --release`, then `wrangler@4 deploy --dry-run`. The workflow's own comment records why the version pin is load-bearing: worker-build must track the `worker` dependency's minor line, and a pre-0.7 toolchain hard-codes a wasm-bindgen CLI version that cannot match what the crate compiled against |
| `secret scan`  | gitleaks, full history                                                                                                                                                                                                                                                                                                                                           |

### `mzizi-registry` — 4 workflows

`ci.yml` jobs, whose names are deliberately bare because the repo ruleset
requires them under those exact strings: **Security Audit**, **Registry
Snapshot**, **Rust**, **Lint**, **Type Check**, **Test**, **Build**.

`lint.yml` jobs are the opposite convention — names carry a `lint /` prefix
(`lint / actionlint`, `lint / JSON validity`, `lint / prettier`,
`lint / markdownlint`, `lint / yamllint`) because GitHub reports the bare
`name:` field to the Checks API and the UI grouping label is not part of it.
That asymmetry inside one repo is a live trap; the file itself documents it.

Also present: `release.yml` (auto-release on a version bump) and
`reusable-ci-vite-plus.yml` — a reusable workflow that lives in the _registry_
repo rather than here. Its header explains why, and the reason is now stale:
see gap 7.

Both `ci.yml` and `lint.yml` carry a `workflow_dispatch` trigger added after
an incident on 2026-08-26 where neither workflow produced any run for
PR `#265` and there was no way to trigger them manually.

Note the trigger difference: `mzizi-registry`'s workflows filter pull requests
on `[main]` only, while the three Rust repos use `[main, "claude/**"]`.

### `agent-tools` — 10 workflows

Private repo. `ci.yml` (`test`, `fundi / wrangler dry-run`), `lint.yml`,
`security.yml` (`pnpm audit --prod --audit-level high` plus a dependency
review, on push, PR and a weekly cron), `docs-check.yml`, `verify-fundi.yml`
(verifies the **live** A2A surface), `auto-assign.yml`,
`deprecate-legacy-npm.yml`, and three `publish-*.yml` workflows for the CLI,
the MCP server and the skills.

`lint.yml` is the only workflow anywhere in this org that calls a reusable
workflow from another repo:
`uses: nyuchi/.github/.github/workflows/reusable-lint.yml@main`.

### `mzizi-site`, `mzizi-docs`, `mzizi-roadmap` — 0 workflows

No CI. `mzizi-site` has no commits at all.

---

## Rust CI: the WASM trap

The single most important thing to know before writing CI for this org.

Two of the three Rust repos ship as WebAssembly: `mzizi-console` runs in a
browser, `mzizi-api-gateway` runs on workerd. **Code can pass every native
check and still fail to compile for the target that actually ships** —
conditional compilation, a host-only dependency, `std` surface that does not
exist on wasm32. A green native `cargo check` is not evidence about the
artefact.

Both repos already handle this, and they handle it _differently_, correctly:

- `mzizi-console` lints on the host and adds
  `cargo check --target wasm32-unknown-unknown --all-targets`.
- `mzizi-api-gateway` runs **clippy itself** against wasm32, because the
  `worker` crate's API is `cfg`'d for that target — a host lint pass there
  would be linting code the Worker never runs.

That is why
[`reusable-rust-ci.yml`](./.github/workflows/reusable-rust-ci.yml) takes both
a `target` input and a separate `clippy-on-target` boolean. One knob would
have forced the two repos onto the same wrong answer.

---

## Reusable workflows published here

A reusable workflow is a `.yml` under `.github/workflows/` with
`on: workflow_call`, called from another repo as
`uses: mzizi-dev/.github/.github/workflows/<name>.yml@main`.

As of 2026-09-11 this repo publishes three:

| Workflow                     | Purpose                                  | Notes                                                                                                                                                                                                              |
| ---------------------------- | ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `reusable-rust-ci.yml`       | fmt / clippy / test / cross-target check | `target` and `clippy-on-target` inputs cover the WASM trap above. `working-directory` covers `mzizi`, whose crate lives in `compiler/`                                                                             |
| `reusable-gitleaks.yml`      | Secret scan                              | Runs the MIT binary directly, **not** `gitleaks/gitleaks-action`, which requires a paid licence for org repos. Defaults to gitleaks 8.21.2 — the version every repo already runs, so adoption changes no behaviour |
| `reusable-pr-title-lint.yml` | Conventional Commits on the PR title     | `amannn/action-semantic-pull-request` pinned by commit SHA                                                                                                                                                         |

**Third-party actions are pinned by commit SHA, not tag.** A tag can be moved
to point at different code; a SHA cannot. `actions/*` are first-party GitHub
and stay on major tags. The pinned SHAs and what they resolved to on
2026-09-11:

| Action                                | SHA        | Resolves to                                                                                                   |
| ------------------------------------- | ---------- | ------------------------------------------------------------------------------------------------------------- |
| `dtolnay/rust-toolchain`              | `6bed076…` | head of the `stable` branch — this action publishes no semver tags, so a branch head is the only thing to pin |
| `Swatinem/rust-cache`                 | `6323deb…` | v2.9.2 (dereferenced from the annotated tag)                                                                  |
| `amannn/action-semantic-pull-request` | `48f2562…` | v6.1.1, which is also where the floating `v6` tag pointed                                                     |

**No repo calls any of these yet.** They are published first so that adopting
one is a small reviewable PR against a single repo, with a real CI run to
compare against, rather than nine repos changing at once. Adoption order that
makes sense: `mzizi-api-gateway` (smallest Rust surface) → `mzizi-console` →
`mzizi`.

### Why not just use `nyuchi/.github`?

`nyuchi/.github` is public and has twenty reusable workflows, including
`reusable-ci-rust-monorepo.yml`, and `agent-tools` already calls its
`reusable-lint.yml` across orgs. So the question is fair, and the answer is
specific rather than territorial — **its Rust workflow has no cross-target
support at all.** Read on 2026-09-11, it has exactly one input (`toolchain`).
There is no `target`, no `wasm32` anywhere in the file, and no
`working-directory`. It runs `cargo clippy --workspace`,
`cargo nextest run --workspace`, `cargo build --workspace --release`,
`cargo doc` and a conditional `cargo deny`, all against the host.

For this org that means:

- **It cannot check either WASM artefact.** The one property that matters
  most for `mzizi-console` and `mzizi-api-gateway` is the one it does not
  test.
- **It assumes a workspace at the repo root.** `mzizi`'s crate is in
  `compiler/`; with no `working-directory` input, `--workspace` from the root
  finds nothing.
- **It requires `cargo nextest`**, which none of these repos use today.
- It sets `RUSTFLAGS: -D warnings` globally, so a warning anywhere fails
  every job rather than the lint job.

It is a good workflow for a root Cargo workspace built for the host. That is
not the shape of any Rust repo here. `reusable-lint.yml` is a genuinely
useful cross-org call and should stay; the Rust one is not a drop-in, and
forking is the honest option rather than the lazy one.

---

## Branch protection and rulesets

**Corrected 2026-09-12. Enforcement now exists.** This section previously said
`GET /orgs/mzizi-dev/rulesets` returns `[]`. It returns two rulesets:

| Ruleset                                 | Scope                                                                    | Enforcement | What it does                                                                                                                                                      |
| --------------------------------------- | ------------------------------------------------------------------------ | ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `org-status-checks` (22953755)          | Org, default branches of `~ALL` repos except `sandbox-*` and `archive-*` | **active**  | Requires the five `lint / *` contexts — `actionlint`, `JSON validity`, `prettier`, `markdownlint`, `yamllint` — with `strict_required_status_checks_policy: true` |
| `enterprise-main-protection` (22953540) | Inherited from the enterprise; not readable on the org endpoint          | `evaluate`  | Reports without blocking                                                                                                                                          |

Two things follow from the active one. **`strict` means a branch must be up to
date with the base before it can merge**, so a PR that sat while `main` moved
needs a rebase before auto-merge will fire. And **the five contexts are exact
strings**: they come from a job called `lint` calling a reusable workflow whose
jobs are named `actionlint`, `JSON validity` and so on, published as
`<caller job> / <called job>`. Converting the caller to a matrix publishes
`lint (actionlint)` instead and satisfies nothing. Copy
`.github/workflows/lint.yml` as-is.

No repo has classic branch protection — every one returns "Branch not
protected". Beneath the org rulesets, one repo-level ruleset exists:

**`mzizi-registry` → ruleset "Default"** (id 14801708, active, no bypass
actors):

| Rule                      | Parameters                                                                                                 |
| ------------------------- | ---------------------------------------------------------------------------------------------------------- |
| `required_linear_history` | —                                                                                                          |
| `pull_request`            | 0 approvals required, review-thread resolution off, `allowed_merge_methods: ["merge", "squash", "rebase"]` |
| `required_status_checks`  | `Lint`, `Type Check`, `Build`, `Security Audit`, `Test`                                                    |

**2026-09-12:** `mzizi-registry` now carries the org's `org-status-checks`
ruleset on top of this one, and its `required_linear_history` rule is no longer
in conflict with the org's merge settings — rebase merges are linear. Its
`allowed_merge_methods: ["merge", "squash", "rebase"]` is now wider than the
repo settings permit, which is harmless: the settings are the narrower gate.

Proposed replacements live in [`github-rulesets/`](./github-rulesets) as
versioned JSON:

- **`org-wide-main-protection.json`** — `deletion`, `non_fast_forward`,
  `required_signatures`, and a `pull_request` rule with
  `allowed_merge_methods: ["merge"]`. Deliberately **no**
  `required_linear_history`. **2026-09-12: both of those choices are now
  wrong.** `["merge"]` names the one method the org has disabled, and
  `required_linear_history` is now the org's actual shape. This file needs
  rewriting to `["rebase"]` before it is ever applied.
- **`release-tag-protection.json`** — makes `v*` tags immutable.

**Neither is applied**, and `org-status-checks` — which _is_ applied — was
created outside this directory, so these two JSON files are no longer a
complete picture of the org's rulesets. Applying an org ruleset changes what
can merge across nine repos at once; that is a human decision, and the apply
command is in each file's `_comment`. Two things to settle before applying:

- `required_signatures` rejects any unsigned commit, including from a bot
  that is not configured to sign. Confirm every author signs, or drop that
  rule in the first pass.
- `required_status_checks` is absent on purpose. A check context that has
  never reported makes every PR permanently unmergeable. Add contexts per
  repo only after reading the exact names off a completed run — the names CI
  emits today are in [What CI actually runs](#what-ci-actually-runs-per-repo).

---

## Repository settings, as they actually are

Re-read on 2026-09-12. Uniform across all nine unless noted.

| Setting                         | Value                                                                                             | Note                                                                                       |
| ------------------------------- | ------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| `allow_rebase_merge`            | **true on all nine**                                                                              | The only permitted method. **Changed 2026-09-12** — was false                              |
| `allow_merge_commit`            | **false on all nine**                                                                             | **Changed 2026-09-12** — was true, and was documented as the only method                   |
| `allow_squash_merge`            | false on all nine                                                                                 | Unchanged                                                                                  |
| `allow_auto_merge`              | **true on all nine**                                                                              | **Changed 2026-09-12** — was true on three. `gh pr merge --rebase --auto` works everywhere |
| `merge_commit_title`            | `MERGE_MESSAGE`                                                                                   | Vestigial — no merge commit can be produced                                                |
| `merge_commit_message`          | `PR_TITLE`                                                                                        | Vestigial, same reason                                                                     |
| `delete_branch_on_merge`        | true on all nine                                                                                  | Already correct — no cleanup needed                                                        |
| `has_wiki`                      | false on `mzizi`, `mzizi-registry`, `mzizi-api-gateway`, `mzizi-site`; **true** on the other five | Unused surface, on by default                                                              |
| Licence                         | Apache-2.0 on eight; **none** on `.github`. **`mzizi-roadmap` now has one**                       | 2026-09-12                                                                                 |
| Secret scanning                 | **enabled on 2 of 8 public repos** — `mzizi-registry`, `mzizi-api-gateway`                        |                                                                                            |
| Secret scanning push protection | Same two                                                                                          |                                                                                            |
| Dependabot security updates     | **disabled on all nine**                                                                          |                                                                                            |
| Private vulnerability reporting | **enabled on 2 of 8** — `mzizi`, `mzizi-registry`. Not available on `agent-tools` (private repo)  | Determines where a security report can actually be filed                                   |

---

## Community-health files

Before this repo was populated, **`mzizi-registry` was the only repo in the
org with any of them**, and every other repo inherited nothing, because this
fallback repo held a one-line README.

`mzizi-registry` has its own `.github/CODEOWNERS`, `SECURITY.md`,
`CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `.github/ISSUE_TEMPLATE/` and
`.github/dependabot.yml`. Its own copies take precedence over anything here,
so two of them being broken (gaps 1 and 2) is not something this repo can fix.

What now applies org-wide by fallback: `.github/PULL_REQUEST_TEMPLATE.md`,
`.github/ISSUE_TEMPLATE/` (bug form, feature form, and a `config.yml` routing
security to a private advisory), `SECURITY.md`, `CONTRIBUTING.md`,
`CODE_OF_CONDUCT.md` (Contributor Covenant 2.1) and `SUPPORT.md`.

**Two things are NOT inheritable, and it is easy to assume otherwise.**

- **CODEOWNERS.** GitHub's list of community health files an org `.github`
  repo can supply as defaults is CODE_OF_CONDUCT.md, CONTRIBUTING.md,
  discussion category forms, FUNDING.yml, issue and PR templates, SECURITY.md
  and SUPPORT.md. CODEOWNERS is not on it, and the CODEOWNERS docs read the
  file from `.github/`, the root, or `docs/` _of the repository_. So
  `.github/CODEOWNERS` in this repo governs this repo and nothing else.
  `CODEOWNERS.example` at the root is the template to copy into each repo.
  This is gap 14.
- **Dependabot.** `.github/dependabot.yml` here covers only this repo;
  `dependabot.example.yml` is the template.

---

## Writing a README

**Added 2026-09-12.** How a repository opens is no longer a per-org question.
The derived, estate-wide standard is `README-STANDARD.md` in the `nyuchi` org's
`.github` repository, proposed in
[`nyuchi/.github#62`](https://github.com/nyuchi/.github/pull/62). It applies to
all seven orgs — `mzizi-dev`, `mukoko-dev`, `nyuchi`, `bundu-labs`,
`shamwari-ai`, `openNTL`, `siafuDB` — and it is derived from the three READMEs
in the estate that already worked, one of which is `mzizi-dev/mzizi-registry`.

**That pull request is open and unmerged**, so the file is not on
`nyuchi/.github`'s default branch and a link to the blob would 404. Read it on
the pull request until it lands.

What it settles, so this document does not have to restate it: a twelve-part
shape, six parts of which are required in every repo however small; the rule
that a README which is wrong is worse than one that is thin; curl every badge
and every URL before committing it; and the Bundu Foundation governance line.
It also carries the branding facts that keep going stale in this org's READMEs
— the palette is **21 colour families** (7 minerals, 7 heritage, 7
experimental), not five and not seven; the architecture is the **DNA double
helix**, and "axis", "axes" and "layer" are retired vocabulary; **there is no
database** behind the registry, which is disk; and `docs.mzizi.dev` does not
resolve and must not be linked.

---

## Known gaps

Verified 2026-09-11. These are documented, not fixed — several are one-line
API calls someone with admin rights should make deliberately, and the rest
are PRs against repos other than this one.

**1. `mzizi-registry`'s own CODEOWNERS assigns nobody.** Every rule in it
names `@nyuchi/core`. That is a team in the `nyuchi` org, not this one — and
`nyuchi` has no `core` team either (its teams are docs, maintainers,
marketing, mukoko, nyuchi-open-projects, platform, security). A team from
another org cannot own code here in any case. Because a repo-local
`CODEOWNERS` is the only kind there is — it is not inheritable from this repo
(gap 14) — the largest repo in the org has no working review routing at all.
**Fix: a PR against `mzizi-registry` replacing `@nyuchi/core` with
`@bryanfawcett`.**

**2. `mzizi-registry`'s `SECURITY.md` points at the old org.** It sends
reporters to `https://github.com/nyuchi/mzizi/security/advisories/new`. The
repo was renamed to `mzizi-dev/mzizi-registry`; GitHub's redirect means the
link happens to resolve, but it names an org that no longer owns the code,
and the version table (4.0.x / 4.1.x) pre-dates the move.

**3. Private vulnerability reporting is off on six of eight public repos.**
Enabled only on `mzizi` and `mzizi-registry`. A reporter following
`SECURITY.md` to `mzizi-console` or `mzizi-api-gateway` finds no private
channel and is pushed toward a public issue — the exact thing the policy
tells them not to do. This repo's `SECURITY.md` works around it by routing
everything to `mzizi`, which is a workaround, not a fix. **Fix:
`gh api -X PUT repos/mzizi-dev/<repo>/private-vulnerability-reporting` per
repo.**

**4. ~~`mzizi-registry`'s ruleset contradicts the merge-only convention.~~
Resolved 2026-09-12 — by the org moving, not by anyone fixing it.** The
"Default" ruleset's `required_linear_history` used to block the org's only
enabled merge method. The org is now rebase-only and rebase merges are linear,
so the rule and the settings agree. The proposed fix recorded here — set
`allowed_merge_methods` to `["merge"]` — would now do the opposite of the right
thing, and `github-rulesets/org-wide-main-protection.json` still contains it.

**5. Secret scanning and push protection are off on six of eight public
repos**, including `mzizi` itself. Push protection stops a secret reaching
the remote; the `gitleaks` CI job only catches what is already committed.
They are complements. `mzizi` and `mzizi-console` currently have the CI half
alone; `mzizi-site`, `mzizi-docs`, `mzizi-roadmap` and `.github` have
neither. Dependabot security updates are off everywhere.

**6. No repo calls the reusable workflows in this repo.** Deliberate for this
pass — publishing and adopting in one change would mean nine repos moving
with no baseline to compare against. Until adoption, each repo's CI stays
duplicated, and a fix to (say) the gitleaks install has to be made three
times. Adoption is tracked as a follow-up per repo.

**7. A reusable workflow lives in `mzizi-registry` on a stale premise.**
`mzizi-registry/.github/workflows/reusable-ci-vite-plus.yml` explains that it
lives there rather than in the org `.github` repo because that repo's "name
begins with a dot, which makes it unattachable to an agent session and so
unmaintainable by the tooling that maintains everything else here". This PR
is a counter-example — the repo is editable. Worth revisiting whether that
workflow should move here, though the migration order it documents (add the
check contexts to rulesets in the right sequence or every PR blocks) is real
and should be followed if it does.

**8. ~~No `required_status_checks` in the proposed org ruleset.~~ Overtaken
2026-09-12.** The concern was real — naming a context that has never reported
blocks every PR — and it was solved in the opposite order to the one proposed
here: the org lint gate (`.github/workflows/lint.yml`) was adopted in every
repo first, and only then was the `org-status-checks` ruleset made active with
the five `lint / *` contexts it emits. The unapplied JSON in
`github-rulesets/` still says no contexts, and no longer describes the org.

**9. Trigger filters are inconsistent.** The three Rust repos filter pull
requests on `[main, "claude/**"]`; `mzizi-registry` filters on `[main]`
alone. Stacked PRs target the branch below them in the stack, so on
`mzizi-registry` every layer above the bottom one currently gets **zero**
checks — and a PR with nothing run is visually indistinguishable from a
passing one.

**10. ~~Three repos have no CI because they have nearly no content.~~ Resolved
2026-09-12.** `mzizi-site` now holds the Astro site that serves the apex,
`mzizi-docs` holds the full Mintlify site, and all nine repos — `mzizi-roadmap`
included — carry the org lint gate. The claim that `mzizi-site` is "completely
empty (no commits)" was true when written and is emphatically not now: it is
the repository serving `mzizi.dev`.

**11. `mzizi-roadmap` is vestigial, its README says so, and the repository is
still not archived.** The fold into `mzizi/design/ROADMAP.md` is complete and
`mzizi-roadmap/README.md` describes itself as archived and read-only.
Re-checked 2026-09-12: `GET /repos/mzizi-dev/mzizi-roadmap` reports
`archived: false`. The repo is public and writable, so the README makes a claim
about GitHub state that GitHub does not agree with. **Fix:
`gh api -X PATCH repos/mzizi-dev/mzizi-roadmap -f archived=true`** — one call,
which also settles `MIGRATION.md` §1's warning against leaving a roadmap living
apart from the code it plans.
**12. `has_wiki` is inconsistent across repos.** Cosmetic, but `has_wiki: true`
on five repos leaves an unused, unwatched surface open on a public org.
**2026-09-12:** `allow_auto_merge` is no longer part of this gap — it is now
true on all nine. `mzizi-roadmap` has gained a licence; `.github` still has
none.

**13. GitHub can enforce SHA-pinning org-wide, and it is switched off.**
`GET /orgs/mzizi-dev/actions/permissions` reports
`sha_pinning_required: false` (with `enabled_repositories: all` and
`allowed_actions: all`). This repo pins its third-party actions by SHA as a
convention, but a convention is only as good as the next contributor's
memory — the org setting makes it a rule that GitHub checks. Turning it on
would first require every existing workflow in the org to be pinned; today
they use floating tags (`actions/checkout@v5`, `pnpm/action-setup@v4`,
`dtolnay/rust-toolchain@stable`), so this is a migration, not a switch. The
same endpoint's `default_workflow_permissions: write` is also worth
revisiting: every workflow in the org starts with a read-write `GITHUB_TOKEN`
unless it narrows its own `permissions`, and `read` would be the safer
default given all five workflows in this repo declare what they need.

**14. No repo has working CODEOWNERS, and this repo cannot fix that
centrally.** CODEOWNERS is not an inheritable community health file (see
[Community-health files](#community-health-files) for the citation), so the
copy in this repo covers only this repo. `mzizi-registry` has the org's only
other CODEOWNERS and it assigns nobody (gap 1). Every other repo has none.
**Fix: copy `CODEOWNERS.example` into `.github/CODEOWNERS` in each repo — one
small PR per repo, eight of them.** Worth doing before
`require_code_owner_review` is ever turned on in a ruleset, because that
setting against a repo with no CODEOWNERS does nothing, and against one with
a broken CODEOWNERS blocks every PR.
