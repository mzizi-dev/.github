# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

`mzizi-dev/.github` is the org-level defaults repo for the `mzizi-dev` GitHub
organisation. It holds no application code: everything here is workflow YAML,
JSON rulesets, lint config and Markdown policy. `ORG_STANDARDS.md` is the
long-form record of what CI and governance actually enforce, so read the
relevant section before changing anything it describes.

## What it provides to the other mzizi-dev repos

| Path | Reaches other repos how |
| --- | --- |
| `profile/README.md` | Rendered as the org page at github.com/mzizi-dev |
| `.github/PULL_REQUEST_TEMPLATE.md`, `.github/ISSUE_TEMPLATE/`, `SECURITY.md`, `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `SUPPORT.md` | GitHub community-health fallback for any repo that lacks its own copy (only `mzizi-registry` ships its own today) |
| `.github/workflows/reusable-*.yml` | Opt-in, via `on: workflow_call` |
| `.github/workflows/org-lint.yml` | Named in the org ruleset's "Require workflows to pass" rule, so it runs on **every PR in every org repo** |
| `workflow-templates/lint.yml` + `.properties.json` | Org starter workflow offered in each repo's Actions tab |
| `CODEOWNERS.example`, `dependabot.example.yml` | Templates to copy. CODEOWNERS and Dependabot have no org fallback; `.github/CODEOWNERS` and `.github/dependabot.yml` here govern this repo only |
| `github-rulesets/*.json` | Proposals applied by hand with `gh api --input`; not applied automatically |

Consumers call the reusable workflows by path and ref:

```yaml
jobs:
  rust:
    uses: mzizi-dev/.github/.github/workflows/reusable-rust-ci.yml@main
    with:
      target: wasm32-unknown-unknown
  secrets:
    uses: mzizi-dev/.github/.github/workflows/reusable-gitleaks.yml@main
```

`reusable-rust-ci.yml` (fmt/clippy/test plus an optional cross-target check,
needed for the WASM repos `mzizi-console` and `mzizi-api-gateway`),
`reusable-gitleaks.yml` (runs the MIT gitleaks binary, not the paid action) and
`reusable-pr-title-lint.yml` (Conventional Commits on the PR title).

## What ripples out when you change things here

- Consumers pin `@main`, so a reusable-workflow change takes effect across the
  org as soon as it reaches `main`. Treat input names and job names as a public
  API: a check publishes as `<caller job> / <called job>`, and rulesets match
  those exact strings. Renaming a job, or turning a caller into a matrix,
  silently breaks required checks.
- `org-lint.yml` runs on every PR org-wide; a broken edit blocks every repo.
  It calls `nyuchi/.github`'s `reusable-lint.yml` and `reusable-vite-plus.yml`
  (prettier and JSON validity are switched off; `vite-plus / fmt` replaces them).
- Edits to the community-health files change what every repo without its own
  copy shows to contributors.
- The Conventional Commit type list exists twice, in `CONTRIBUTING.md` and
  `reusable-pr-title-lint.yml`. Change both together.
- Third-party actions are pinned by commit SHA; first-party `actions/*` stay on
  major tags. Keep that split when adding or bumping actions.

## Commands

CI (`.github/workflows/ci.yml`) runs actionlint, a JSON/required-keys check of
`github-rulesets/*.json`, and `reusable-gitleaks.yml` by local path (`uses:
./...`) so a PR exercises its own copy. Org lint adds markdownlint and
yamllint. Locally:

```sh
yamllint -c .yamllint.yaml .
actionlint                                   # binary from rhysd/actionlint releases; CI pins 1.7.12
for f in github-rulesets/*.json; do python3 -m json.tool "$f" >/dev/null || echo "bad: $f"; done
```

The lint configs at the root (`.yamllint.yaml`, `.markdownlint.jsonc`,
`.prettierrc`, `.prettierignore`) are copies of the canonical ones in
`nyuchi/.github`; a repo's own copy wins over the fallback.

## Branches, releases and merging

- Work branches off `staging` and open PRs into `staging`. Each push to
  `staging` is tagged as the next patch (`staging-version.yml`); a successful CI
  run on a push to `main` tags the next minor and creates a release
  (`main-release.yml`). Both depend on `nyuchi/.github` pinned by SHA.
- Branch prefixes: `feat/`, `fix/`, `docs/`, or `claude/` for agent work. PR
  filters include `"claude/**"` so stacked PRs still get checks.
- PR titles are Conventional Commits: lowercase, imperative subject, no
  trailing period (a subject starting with an all-caps word fails).
- The org is **rebase-only** (`gh pr merge <n> --rebase --auto`, never
  `--admin`); every commit lands on `main` as written. `README.md` and
  `ORG_STANDARDS.md` reflect this. `CONTRIBUTING.md` and the PR template still
  say merge-only and are stale on that point.
- Never link `docs.mzizi.dev` in new prose (it does not resolve); see
  `ORG_STANDARDS.md#writing-a-readme` for branding facts READMEs get wrong.
