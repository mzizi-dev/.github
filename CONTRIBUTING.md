# Contributing to mzizi-dev

This is the org-wide fallback guide. A repo with its own `CONTRIBUTING.md`
overrides it — today only `mzizi-registry` has one.

## The one thing that is different here

**This org is merge-only.** Squash merging and rebase merging are disabled on
all nine repos. This is deliberate, not an oversight, and the reason is
written down in `mzizi-dev/mzizi`'s `MIGRATION.md` §1.1:

| Allow merge commits | **yes** | The ecosystem convention is merge-only; history stays truthful |
| Allow squash merging | **no** | Squash discards the per-commit reasoning this project depends on |
| Allow rebase merging | **no** | Same |

So: **your commits land on `main` exactly as you wrote them, and stay there.**
That changes how you should work.

- Write each commit as a coherent step, with a message that explains *why*.
  Nobody is going to squash your "wip" and "fix typo" commits away for you.
- Clean the branch up before requesting review — `git rebase -i` on your own
  branch, not on `main`.
- Do not merge `main` into your branch to resolve a conflict. Rebase your
  branch onto `main`, then let the PR merge produce the single merge commit.
- Merge with `gh pr merge <n> --merge --delete-branch`. `--squash` and
  `--rebase` will be rejected by the repo settings.

The merge commit reads:

```
Merge pull request #12 from mzizi-dev/claude/org-defaults

feat(ci): add the rust reusable workflow
```

Every repo is configured `merge_commit_title=MERGE_MESSAGE`,
`merge_commit_message=PR_TITLE`, so the PR title is the body — the one line
that summarises your whole branch in `git log --first-parent`. Write it as
carefully as a commit subject.

## Branches

Use a prefix that says who or what is doing the work: `feat/`, `fix/`,
`docs/`, or `claude/` for agent-authored branches. Branches are deleted on
merge (`delete_branch_on_merge` is true on all nine repos).

## PR titles — Conventional Commits

The PR title must parse as a Conventional Commit. Allowed types:

`feat` · `fix` · `perf` · `refactor` · `docs` · `test` · `build` · `ci` ·
`chore` · `revert` · `style`

Rules:

- Subject starts **lowercase**.
- Subject is **imperative** — "add", not "adds" or "added".
- No trailing period.
- Scope is optional: `feat(compiler): ...` is fine, `feat: ...` is fine.
- A title literally starting with `wip` is exempt. That is **not** the same
  as marking the PR a draft — a draft PR with an ordinary title is still
  checked and can still fail.

This is enforced by
[`reusable-pr-title-lint.yml`](./.github/workflows/reusable-pr-title-lint.yml)
in repos that call it. See ORG_STANDARDS.md for which repos do today (the
honest answer right now: none).

If you change the type list here, change it in that workflow too. They are
two copies of one decision.

## Rust

Most of this org is Rust. Before you push:

```sh
cargo fmt --all -- --check
cargo clippy --all-targets -- -D warnings
cargo test
```

**If the crate ships as WASM, that is not enough.** `mzizi-console` runs in a
browser and `mzizi-api-gateway` runs on workerd; both compile to
`wasm32-unknown-unknown`. Code can pass every command above and still fail to
build for the target that actually ships:

```sh
rustup target add wasm32-unknown-unknown
cargo check --target wasm32-unknown-unknown --all-targets
```

For `mzizi-api-gateway` specifically, run clippy against the target too — the
`worker` crate's API is `cfg`'d for wasm32, so a host lint pass checks code
the Worker never runs:

```sh
cargo clippy --target wasm32-unknown-unknown --all-targets -- -D warnings
```

## CI triggers, if you are adding a workflow

Filter pull requests on `[main, "claude/**"]`, not `[main]` alone:

```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main, "claude/**"]
```

Stacked PRs target the branch below them in the stack. A filter of `[main]`
gives every layer but the bottom one **zero** checks — and a PR with nothing
run looks identical to a passing one in the UI. The existing `ci.yml` in
`mzizi`, `mzizi-console` and `mzizi-api-gateway` all carry this filter and a
comment explaining it.

## Secrets

Never commit a key, token or `.env`. `gitleaks` runs in CI on `mzizi`,
`mzizi-console` and `mzizi-api-gateway` and scans **full history**, so a
secret committed and then removed in a later commit still fails the build —
correctly. If that happens, rotate the credential first; rewriting history is
not a fix on its own.

## Reporting a security issue

Not through a PR or an issue. See [SECURITY.md](./SECURITY.md).

## Where to ask

[SUPPORT.md](./SUPPORT.md).

## Licence

Contributions are made under each repo's licence — Apache-2.0 for everything
that has one today. Mzizi framework, components and compiler logic are Bundu
Foundation IP (`mzizi/CHARTER.md`).
