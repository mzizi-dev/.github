# mzizi-dev/.github

> Org-wide defaults for the [`mzizi-dev`](https://github.com/mzizi-dev) GitHub organisation — community-health files, reusable workflows, and the record of what CI and governance actually enforce.

[![CI](https://github.com/mzizi-dev/.github/actions/workflows/ci.yml/badge.svg)](https://github.com/mzizi-dev/.github/actions/workflows/ci.yml)
[![Lint](https://github.com/mzizi-dev/.github/actions/workflows/lint.yml/badge.svg)](https://github.com/mzizi-dev/.github/actions/workflows/lint.yml)

**Org:** nine repositories | **Merge method:** squash, through the merge queue | **Standard:** [ORG_STANDARDS.md](./ORG_STANDARDS.md)

GitHub falls back to what lives here for any repo in the org that does not ship
its own copy, and this is where the org's reusable workflows live.

**Start here: [ORG_STANDARDS.md](./ORG_STANDARDS.md)** — what CI actually runs
in each repo today, and an explicit list of what does not exist yet.

## What is in here

| Path                                                                                                                                                  | Applies to                                                                                           |
| ----------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| [`.github/CODEOWNERS`](./.github/CODEOWNERS)                                                                                                          | **This repo only.** CODEOWNERS is not inheritable — see [`CODEOWNERS.example`](./CODEOWNERS.example) |
| [`profile/README.md`](./profile/README.md)                                                                                                            | The org page at <https://github.com/mzizi-dev>                                                       |
| [`.github/PULL_REQUEST_TEMPLATE.md`](./.github/PULL_REQUEST_TEMPLATE.md)                                                                              | Every repo without its own                                                                           |
| [`.github/ISSUE_TEMPLATE/`](./.github/ISSUE_TEMPLATE)                                                                                                 | Every repo without its own                                                                           |
| [`SECURITY.md`](./SECURITY.md) · [`CONTRIBUTING.md`](./CONTRIBUTING.md) · [`CODE_OF_CONDUCT.md`](./CODE_OF_CONDUCT.md) · [`SUPPORT.md`](./SUPPORT.md) | Every repo without its own                                                                           |
| [`.github/workflows/reusable-*.yml`](./.github/workflows)                                                                                             | Any repo that calls them — opt-in, per repo                                                          |
| [`github-rulesets/`](./github-rulesets)                                                                                                               | Proposals. **Not applied.**                                                                          |
| [`CODEOWNERS.example`](./CODEOWNERS.example) · [`dependabot.example.yml`](./dependabot.example.yml)                                                   | Templates to copy — neither CODEOWNERS nor Dependabot has an org-wide fallback                       |

## Reusable workflows

```yaml
jobs:
  rust:
    uses: mzizi-dev/.github/.github/workflows/reusable-rust-ci.yml@main
    with:
      target: wasm32-unknown-unknown
  secrets:
    uses: mzizi-dev/.github/.github/workflows/reusable-gitleaks.yml@main
```

| Workflow                                                                       | What it does                                                                                                                                                    |
| ------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`reusable-rust-ci.yml`](./.github/workflows/reusable-rust-ci.yml)             | fmt, clippy, test, and an optional check against a second target — `wasm32-unknown-unknown` is the one that matters for `mzizi-console` and `mzizi-api-gateway` |
| [`reusable-gitleaks.yml`](./.github/workflows/reusable-gitleaks.yml)           | Secret scan, running the MIT-licensed binary directly rather than the paid-licence wrapper action                                                               |
| [`reusable-pr-title-lint.yml`](./.github/workflows/reusable-pr-title-lint.yml) | Conventional Commits on the PR title                                                                                                                            |

No repo calls these yet — they are published here first so adoption is a
reviewable PR per repo rather than a big-bang change. See ORG_STANDARDS.md.

## This org squash-merges through a merge queue

**Changed 2026-10-21, with the owner's approval. This section previously said
rebase-only.**

Squash merges through the merge queue. Rebase and merge commits are disabled
(repo settings and rulesets, 2026-10-21). Verified on every repo:

```text
allow_squash_merge   true       (was: false)
allow_rebase_merge   false      (was: true)
allow_merge_commit   false      (unchanged)
allow_auto_merge     true       (unchanged)
squash message       PR title + commit details (COMMIT_MESSAGES)
merge queue          SQUASH, on staging and main
```

```sh
gh pr merge <n> --squash --auto
```

`--auto` adds the PR to the merge queue. Never `--admin`. Merged branches are
deleted.

**The squash keeps every commit message.** `mzizi/MIGRATION.md` §1.1 — _"squash
discards the per-commit reasoning this project depends on"_ — argued against a
squash that keeps only the title. The squash commit here keeps every commit
message in its body (`COMMIT_MESSAGES`), so the per-commit reasoning is
preserved. Write your commits for the person reading them in a year.

"Require branches to be up to date" is off. Updating a branch with the base
(GitHub's "Update branch") is fine; the squash makes `main` linear. The merge
queue tests each PR against the latest base.
[CONTRIBUTING.md](./CONTRIBUTING.md) and
[ORG_STANDARDS.md](./ORG_STANDARDS.md#the-merge-convention) have the rest.

## How a README in this org should open

The estate-wide README standard is `README-STANDARD.md` in the `nyuchi` org's
`.github` repository, proposed in
[`nyuchi/.github#62`](https://github.com/nyuchi/.github/pull/62). It covers all
seven orgs — `mzizi-dev`, `mukoko-dev`, `nyuchi`, `bundu-labs`, `shamwari-ai`,
`openNTL`, `siafuDB` — and it is derived from three READMEs in the estate that
already worked, one of them `mzizi-dev/mzizi-registry`.

**That pull request is open and unmerged**, so the file is not on
`nyuchi/.github`'s default branch yet and a link to the blob would 404. Read it
on the pull request.

It also carries the branding facts this org's READMEs keep getting wrong: the
palette is **21 colour families** — 7 minerals, 7 heritage, 7 experimental —
not five and not seven; the architecture is the **DNA double helix** and
"axis", "axes" and "layer" are retired vocabulary; **there is no database**
behind the registry, which is disk; and `docs.mzizi.dev` does not resolve and
must not be linked. [ORG_STANDARDS.md](./ORG_STANDARDS.md#writing-a-readme)
summarises it.

## Licence

This repository carries no licence file. Everything it holds is org
configuration and policy boilerplate rather than code.

Mzizi is an open-architecture project, owned and operated by **Mzizi**.
**Nyuchi** operates the revenue products built on it — the Mzizi console,
Fundi, paid plans and billing.
