# Org CI and governance standards — `mzizi-dev`

**Everything here was read off the GitHub API on 2026-09-11**, not inferred
from what ought to be true. Where something is a proposal rather than
something running today, it is in [Known gaps](#known-gaps) and says so.
Nothing on this page should be read as enforced unless it says it is.

If you are adding CI to a repo in this org, the two sections worth reading
first are [The merge-only convention](#the-merge-only-convention) and
[Rust CI: the WASM trap](#rust-ci-the-wasm-trap).

---

## The org

Nine repos. Two members — `@bryanfawcett` (admin) and `@michellellawson`
(member). **No teams exist**, which is why `.github/CODEOWNERS` names users
rather than a `@mzizi-dev/...` handle.

| Repo | Public | Stack | State |
|---|---|---|---|
| [`mzizi`](https://github.com/mzizi-dev/mzizi) | yes | Rust | The language, compiler and `mz` CLI. Bundu Foundation IP. Has content and CI |
| [`mzizi-registry`](https://github.com/mzizi-dev/mzizi-registry) | yes | Next.js (+ Rust crates) on Vercel | The component registry and mzizi.dev. The largest repo, and the only one with its own community-health files |
| [`mzizi-console`](https://github.com/mzizi-dev/mzizi-console) | yes | Astro + Rust/Dioxus WASM islands | app.mzizi.dev. Has content and CI |
| [`mzizi-api-gateway`](https://github.com/mzizi-dev/mzizi-api-gateway) | yes | Pure-Rust Cloudflare Worker (workers-rs) | api.mzizi.dev. Has content and CI |
| [`mzizi-site`](https://github.com/mzizi-dev/mzizi-site) | yes | — | **Completely empty** — no commits at all. Not "a README": the API returns "This repository is empty" |
| [`mzizi-docs`](https://github.com/mzizi-dev/mzizi-docs) | yes | Mintlify (planned) | README and LICENSE only. No `.github/` directory, no CI |
| [`mzizi-roadmap`](https://github.com/mzizi-dev/mzizi-roadmap) | yes | — | README only. `mzizi`'s own history shows the roadmap being folded into `mzizi/design/ROADMAP.md`, so this repo may be vestigial |
| [`agent-tools`](https://github.com/mzizi-dev/agent-tools) | **no** | TypeScript / pnpm | MCP server, `fundi` agent, CLI, skills. The most CI of any repo — ten workflows |
| [`.github`](https://github.com/mzizi-dev/.github) | yes | — | This repo |

---

## The merge-only convention

**Squash merging and rebase merging are disabled on all nine repos.
`allow_merge_commit` is true everywhere; `allow_squash_merge` and
`allow_rebase_merge` are false everywhere.** Verified on each repo
individually.

This is a decision, not a default. `mzizi-dev/mzizi`'s `MIGRATION.md` §1.1:

| Allow merge commits | **yes** | The ecosystem convention is merge-only; history stays truthful |
| Allow squash merging | **no** | Squash discards the per-commit reasoning this project depends on |
| Allow rebase merging | **no** | Same |

It is visible in the history. `mzizi`, `mzizi-api-gateway` and `agent-tools`
all have two-parent `Merge pull request #N from mzizi-dev/...` commits at or
near the tip of `main`.

Three consequences that are easy to get wrong:

**1. Your commits are permanent, exactly as written.** Nothing folds them
together. Clean the branch up with an interactive rebase *on your own branch*
before requesting review, and write messages that explain why.

**2. The PR title becomes the merge commit body, not its subject.** Every
repo is `merge_commit_title=MERGE_MESSAGE`, `merge_commit_message=PR_TITLE`
(verified on all nine). The result:

```
Merge pull request #12 from mzizi-dev/claude/org-defaults

feat(ci): add the rust reusable workflow
```

So a PR-title lint here is not the squash-org version of the same check. It
is not guarding the commit subject on `main` — it is guarding the one line
that summarises the branch in `git log --first-parent`. Worth having, and
worth being precise about.

**3. `required_linear_history` is incompatible with this org.** That ruleset
rule blocks merge commits, and a merge commit is the only merge this org
permits. The two together make every PR unmergeable. This is not theoretical
— see gap 4.

Merge with:

```sh
gh pr merge <n> --merge --delete-branch
```

`--squash` and `--rebase` are rejected by the repo settings.

---

## What CI actually runs, per repo

Read from the workflow files in each repo's default branch.

### `mzizi` — 2 workflows

`ci.yml` triggers on push to `main` and pull requests to `main` **or**
`claude/**`, with `concurrency` cancelling superseded runs off `main`.

| Job (check name) | What it runs |
|---|---|
| `compiler` | `cargo fmt -- --check`, `cargo clippy --all-targets -- -D warnings`, `cargo test`, all in `compiler/` |
| `compiler` (cont.) | `mz check ../examples/connectivity_bar.mz`, then `mz check` over **every** file in `primitives/`, using the built binary rather than the test harness |
| `secret scan` | `gitleaks detect` at `fetch-depth: 0` — full history, because this repo arrived via `git subtree split` and every commit reached CI for the first time at once |

`mzizi-lang-benchmark-dispatch.yml` is the second workflow. On push to
`main` it dispatches to a held-out benchmark runner named by the
`MZIZI_HELDOUT_REPO` repo variable, guarded by
`if: github.repository == 'mzizi-dev/mzizi'` so a fork cannot fire it. The
held-out task set is deliberately kept out of the public repo — per
`MIGRATION.md` §5, it is "withheld so the benchmark measures the language
rather than memorisation". Nothing here checks whether that runner exists.

The `mz check` steps are the interesting part and are worth copying in
spirit: they assert the *shipped binary* still accepts the corpus, which is a
different claim from "the tests pass".

### `mzizi-console` — 1 workflow

| Job | What it runs |
|---|---|
| `rust` | fmt, clippy (host), `cargo test`, **and `cargo check --target wasm32-unknown-unknown --all-targets`** |
| `web` | pnpm 10.33.0, Node 22, `astro check`, `astro build`. The build is what proves the two toolchains compose — the island script references a bundle name derived from the crate name, so a rename that updates one and not the other fails here instead of serving a blank page |
| `secret scan` | gitleaks, full history |

### `mzizi-api-gateway` — 1 workflow

| Job | What it runs |
|---|---|
| `rust` | fmt; **clippy against `wasm32-unknown-unknown`**, not the host, because the `worker` crate's API is `cfg`'d for that target; `cargo check --target wasm32-unknown-unknown`; `cargo test` |
| `worker build` | `cargo install worker-build --version ^0.8`, `worker-build --release`, then `wrangler@4 deploy --dry-run`. The workflow's own comment records why the version pin is load-bearing: worker-build must track the `worker` dependency's minor line, and a pre-0.7 toolchain hard-codes a wasm-bindgen CLI version that cannot match what the crate compiled against |
| `secret scan` | gitleaks, full history |

### `mzizi-registry` — 4 workflows

`ci.yml` jobs, whose names are deliberately bare because the repo ruleset
requires them under those exact strings: **Security Audit**, **Registry
Snapshot**, **Rust**, **Lint**, **Type Check**, **Test**, **Build**.

`lint.yml` jobs are the opposite convention — names carry a `lint / ` prefix
(`lint / actionlint`, `lint / JSON validity`, `lint / prettier`,
`lint / markdownlint`, `lint / yamllint`) because GitHub reports the bare
`name:` field to the Checks API and the UI grouping label is not part of it.
That asymmetry inside one repo is a live trap; the file itself documents it.

Also present: `release.yml` (auto-release on a version bump) and
`reusable-ci-vite-plus.yml` — a reusable workflow that lives in the *registry*
repo rather than here. Its header explains why, and the reason is now stale:
see gap 7.

Both `ci.yml` and `lint.yml` carry a `workflow_dispatch` trigger added after
an incident on 2026-08-26 where neither workflow produced any run for PR
#265 and there was no way to trigger them manually.

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

Both repos already handle this, and they handle it *differently*, correctly:

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

| Workflow | Purpose | Notes |
|---|---|---|
| `reusable-rust-ci.yml` | fmt / clippy / test / cross-target check | `target` and `clippy-on-target` inputs cover the WASM trap above. `working-directory` covers `mzizi`, whose crate lives in `compiler/` |
| `reusable-gitleaks.yml` | Secret scan | Runs the MIT binary directly, **not** `gitleaks/gitleaks-action`, which requires a paid licence for org repos. Defaults to gitleaks 8.21.2 — the version every repo already runs, so adoption changes no behaviour |
| `reusable-pr-title-lint.yml` | Conventional Commits on the PR title | `amannn/action-semantic-pull-request` pinned by commit SHA |

**Third-party actions are pinned by commit SHA, not tag.** A tag can be moved
to point at different code; a SHA cannot. `actions/*` are first-party GitHub
and stay on major tags. The pinned SHAs and what they resolved to on
2026-09-11:

| Action | SHA | Resolves to |
|---|---|---|
| `dtolnay/rust-toolchain` | `6bed076…` | head of the `stable` branch — this action publishes no semver tags, so a branch head is the only thing to pin |
| `Swatinem/rust-cache` | `6323deb…` | v2.9.2 (dereferenced from the annotated tag) |
| `amannn/action-semantic-pull-request` | `48f2562…` | v6.1.1, which is also where the floating `v6` tag pointed |

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

**Nothing is enforced today.** `GET /orgs/mzizi-dev/rulesets` returns `[]`.
No repo has classic branch protection — every one returns "Branch not
protected". There is exactly one ruleset in the entire org, and it is
repo-level:

**`mzizi-registry` → ruleset "Default"** (id 14801708, active, no bypass
actors):

| Rule | Parameters |
|---|---|
| `required_linear_history` | — |
| `pull_request` | 0 approvals required, review-thread resolution off, `allowed_merge_methods: ["merge", "squash", "rebase"]` |
| `required_status_checks` | `Lint`, `Type Check`, `Build`, `Security Audit`, `Test` |

That ruleset is the org's only worked example of required status checks, and
it is also broken — see gap 4.

Proposed replacements live in [`github-rulesets/`](./github-rulesets) as
versioned JSON:

- **`org-wide-main-protection.json`** — `deletion`, `non_fast_forward`,
  `required_signatures`, and a `pull_request` rule with
  `allowed_merge_methods: ["merge"]`. Deliberately **no**
  `required_linear_history`, for the reason above.
- **`release-tag-protection.json`** — makes `v*` tags immutable.

**Neither is applied.** Applying an org ruleset changes what can merge across
nine repos at once; that is a human decision, and the apply command is in
each file's `_comment`. Two things to settle before applying:

- `required_signatures` rejects any unsigned commit, including from a bot
  that is not configured to sign. Confirm every author signs, or drop that
  rule in the first pass.
- `required_status_checks` is absent on purpose. A check context that has
  never reported makes every PR permanently unmergeable. Add contexts per
  repo only after reading the exact names off a completed run — the names CI
  emits today are in [What CI actually runs](#what-ci-actually-runs-per-repo).

---

## Repository settings, as they actually are

Uniform across all nine unless noted.

| Setting | Value | Note |
|---|---|---|
| `allow_merge_commit` | true | The only permitted method |
| `allow_squash_merge` / `allow_rebase_merge` | false | MIGRATION.md §1.1 |
| `merge_commit_title` | `MERGE_MESSAGE` | "Merge pull request #N from …" |
| `merge_commit_message` | `PR_TITLE` | The PR title becomes the body |
| `delete_branch_on_merge` | **true** on all nine | Already correct — no cleanup needed |
| `allow_auto_merge` | true on `mzizi-registry`, `mzizi-api-gateway`, `agent-tools`; **false** on the other six | Inconsistent |
| `has_wiki` | false on `mzizi`, `mzizi-registry`, `mzizi-api-gateway`, `mzizi-site`; **true** on the other five | Unused surface, on by default |
| Licence | Apache-2.0 on seven; **none** on `mzizi-roadmap` and `.github` | |
| Secret scanning | **enabled on 2 of 8 public repos** — `mzizi-registry`, `mzizi-api-gateway` | |
| Secret scanning push protection | Same two | |
| Dependabot security updates | **disabled on all nine** | |
| Private vulnerability reporting | **enabled on 2 of 8** — `mzizi`, `mzizi-registry`. Not available on `agent-tools` (private repo) | Determines where a security report can actually be filed |

---

## Community-health files

Before this repo was populated, **`mzizi-registry` was the only repo in the
org with any of them**, and every other repo inherited nothing, because this
fallback repo held a one-line README.

`mzizi-registry` has its own `.github/CODEOWNERS`, `SECURITY.md`,
`CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `.github/ISSUE_TEMPLATE/` and
`.github/dependabot.yml`. Its own copies take precedence over anything here,
so two of them being broken (gaps 1 and 2) is not something this repo can fix.

What now applies org-wide by fallback: `.github/CODEOWNERS`,
`.github/PULL_REQUEST_TEMPLATE.md`, `.github/ISSUE_TEMPLATE/` (bug form,
feature form, and a `config.yml` routing security to a private advisory),
`SECURITY.md`, `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md` (Contributor Covenant
2.1) and `SUPPORT.md`.

**Dependabot does not have an org-wide fallback.** `.github/dependabot.yml`
here covers only this repo; `dependabot.example.yml` is a template to copy.

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
`CODEOWNERS` takes precedence, the largest repo in the org is the one this
repo's `CODEOWNERS` does not reach. **Fix: a PR against `mzizi-registry`
replacing `@nyuchi/core` with `@bryanfawcett`.**

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

**4. `mzizi-registry`'s ruleset contradicts the merge-only convention.** Its
"Default" ruleset requires `required_linear_history`, which blocks merge
commits — while the repo's only enabled merge method *is* the merge commit.
The next PR merged there with the merge button should be rejected by the
ruleset. Stated as a prediction rather than an observation, honestly: the
last merges on that repo (PRs #317, #318, 2026-09-08) produced single-parent
commits in squash format, so they pre-date the merge-only settings and no
merge commit has been attempted against the rule yet. **Fix: drop
`required_linear_history` from that ruleset and set `allowed_merge_methods`
to `["merge"]`.**

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

**8. No `required_status_checks` in the proposed org ruleset.** Deliberate —
naming a context that has never reported blocks every PR. Add per repo after
CI has run once and the exact names are readable off a completed run.

**9. Trigger filters are inconsistent.** The three Rust repos filter pull
requests on `[main, "claude/**"]`; `mzizi-registry` filters on `[main]`
alone. Stacked PRs target the branch below them in the stack, so on
`mzizi-registry` every layer above the bottom one currently gets **zero**
checks — and a PR with nothing run is visually indistinguishable from a
passing one.

**10. Three repos have no CI because they have nearly no content.**
`mzizi-site` is completely empty (no commits); `mzizi-docs` and
`mzizi-roadmap` hold a README. Nothing to fix until there is something to
check — noted so the absence is not mistaken for an oversight.

**11. `mzizi-roadmap` may be vestigial.** `mzizi`'s history contains
`docs: fold mzizi-dev/mzizi-roadmap into design/ROADMAP.md` (PR #3, merged).
If the fold is complete, the repo should be archived rather than left as a
second place a roadmap might live — which `MIGRATION.md` §1 explicitly warns
against: "do not leave a roadmap living apart from the code it plans."

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

**12. `allow_auto_merge` and `has_wiki` are inconsistent across repos.**
Cosmetic, but `has_wiki: true` on five repos leaves an unused, unwatched
surface open on a public org. `mzizi-roadmap` and `.github` also carry no
licence.
