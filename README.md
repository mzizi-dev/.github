# mzizi-dev/.github

Org-wide defaults for the [`mzizi-dev`](https://github.com/mzizi-dev) GitHub
organisation. GitHub falls back to what lives here for any repo in the org
that does not ship its own copy, and this is where the org's reusable
workflows live.

**Start here: [ORG_STANDARDS.md](./ORG_STANDARDS.md)** — what CI actually runs
in each repo today, and an explicit list of what does not exist yet.

## What is in here

| Path | Applies to |
|---|---|
| [`.github/CODEOWNERS`](./.github/CODEOWNERS) | **This repo only.** CODEOWNERS is not inheritable — see [`CODEOWNERS.example`](./CODEOWNERS.example) |
| [`.github/PULL_REQUEST_TEMPLATE.md`](./.github/PULL_REQUEST_TEMPLATE.md) | Every repo without its own |
| [`.github/ISSUE_TEMPLATE/`](./.github/ISSUE_TEMPLATE) | Every repo without its own |
| [`SECURITY.md`](./SECURITY.md) · [`CONTRIBUTING.md`](./CONTRIBUTING.md) · [`CODE_OF_CONDUCT.md`](./CODE_OF_CONDUCT.md) · [`SUPPORT.md`](./SUPPORT.md) | Every repo without its own |
| [`.github/workflows/reusable-*.yml`](./.github/workflows) | Any repo that calls them — opt-in, per repo |
| [`github-rulesets/`](./github-rulesets) | Proposals. **Not applied.** |
| [`CODEOWNERS.example`](./CODEOWNERS.example) · [`dependabot.example.yml`](./dependabot.example.yml) | Templates to copy — neither CODEOWNERS nor Dependabot has an org-wide fallback |

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

| Workflow | What it does |
|---|---|
| [`reusable-rust-ci.yml`](./.github/workflows/reusable-rust-ci.yml) | fmt, clippy, test, and an optional check against a second target — `wasm32-unknown-unknown` is the one that matters for `mzizi-console` and `mzizi-api-gateway` |
| [`reusable-gitleaks.yml`](./.github/workflows/reusable-gitleaks.yml) | Secret scan, running the MIT-licensed binary directly rather than the paid-licence wrapper action |
| [`reusable-pr-title-lint.yml`](./.github/workflows/reusable-pr-title-lint.yml) | Conventional Commits on the PR title |

No repo calls these yet — they are published here first so adoption is a
reviewable PR per repo rather than a big-bang change. See ORG_STANDARDS.md.

## This org is merge-only

Squash and rebase merging are disabled on all nine repos, deliberately:
`mzizi/MIGRATION.md` §1.1 — *"Squash discards the per-commit reasoning this
project depends on."*

Merge with `gh pr merge <n> --merge --delete-branch`. Write your commits for
the person reading them in a year; they are not going to be squashed away.
[CONTRIBUTING.md](./CONTRIBUTING.md) has the rest.
