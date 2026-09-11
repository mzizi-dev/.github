## What this changes

<!-- One or two sentences. The Conventional Commit type lives in the PR
     title; this is the part a reviewer reads first. -->

## Why

<!-- The reasoning the diff cannot express. If it closes an issue, say
     "Closes #123". If it follows another PR, link it.

     This section matters more here than in most orgs. mzizi-dev is
     MERGE-ONLY — squash and rebase merging are disabled on all nine repos,
     because "squash discards the per-commit reasoning this project depends
     on" (mzizi/MIGRATION.md §1.1). Your individual commits survive on main
     forever. Write them, and this, for the person reading them in a year. -->

## Commits

- [ ] Each commit is a coherent step with a message that says _why_, not
      just _what_. They are not going to be squashed away.
- [ ] No "fix typo" / "address review" commits left in the history — fold
      them into the commit they belong to before requesting review.

## Checks

- [ ] CI is green on this PR.
- [ ] PR title follows Conventional Commits (`feat:`, `fix:`, `docs:`, …),
      subject lowercase and imperative, no trailing period.
- [ ] `cargo fmt --check` and `cargo clippy -- -D warnings` pass locally for
      any crate touched.

## WASM

<!-- Delete if this touches no Rust that ships to a browser or to workerd. -->

- [ ] `cargo check --target wasm32-unknown-unknown --all-targets` passes.

> `mzizi-console` ships as WASM in the browser and `mzizi-api-gateway` ships
> as WASM on workerd. Code can pass a native `cargo check` and still fail to
> compile for either. A native-only pass is not evidence.

## Deployment

<!-- Does this need a `wrangler deploy`, a secret set, a DNS change, or a
     version bump anywhere downstream? "No" is a fine answer — say it
     explicitly rather than leaving the section blank. -->
