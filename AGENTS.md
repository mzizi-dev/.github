# AGENTS.md

Guidance for every AI coding agent (Claude Code, Cursor, Copilot and the
rest) working in this repository. Org `.github` repos keep agent rules in
`AGENTS.md` only: no `CLAUDE.md` and no `.claude/` directory. The
enterprise-wide agent rules are in
[`nyuchi/.github/AGENTS.md`](https://github.com/nyuchi/.github/blob/main/AGENTS.md);
this file adds what is specific to `mzizi-dev`.

## Mission (owner, 2026-10-24)

Read this first. It applies to every repository in the Bundu ecosystem, and
every decision is judged against the whole platform, not only the repo in
front of you.

**What we are building.** A platform built to compete with the top five
platforms in the world. It is built from Africa, but not as "a product for
Africa". By default it is open data on published open standards, with no
proprietary data structures, so any system can connect to our data,
transformations and services.

**The ecosystem.** Read nyuchi.com, bundu.org, mukoko.com and mzizi.dev, and
their `llms.txt` files.

- **Bundu Foundation** governs the standards and doctrine. Ubuntu is the spec:
  survival, solidarity, compassion, respect, dignity. Personal data stays on
  the user's device or pod, under the user's keys. No surveillance business
  model. The data commons is open.
- **Nyuchi Africa** is the frontier infrastructure (API Platform, Pay, Web
  Services, Weather Stations, Logistics and more), with the console at
  platform.nyuchi.com.
- **Mukoko** is a privacy-first consumer super-app: one identity, one payments
  rail, offline-first sync, and English, Shona and Ndebele at parity. Kweli is
  its trust and verification layer.
- **Mzizi** is the open architecture framework and language, built on Rust.
  Mzizi is the rule: no app drifts from it, and filling a design-system gap
  so an app conforms to Mzizi needs no approval.
- **Four gateways** (nyuchi-api#353), each rebuilt in Rust on the same layout,
  core, origin and edge (nyuchi-api#355):
  - Mzizi API: public.
  - Nyuchi API: B2B, api.nyuchi.com.
  - Mukoko API: consumer.
  - Nyuchi Servers API: internal, no public IP.

**The engineering bar**, for every service and data model:

1. **Open standards by default** (nyuchi-api#369). Never invent a format when
   a published standard exists:
   - schema.org types with JSON-LD; GeoJSON (RFC 7946) for places;
   - ISO 3166 for countries, ISO 4217 for currencies, RFC 3339 for times,
     BCP 47 for languages (`en`, `sn`, `nd`);
   - CloudEvents for events and DCAT for open datasets;
   - OpenAPI 3.1 for every API;
   - RFC 9457 Problem Details for every error;
   - OAuth 2.1 and OIDC, with RFC 9728 protected-resource metadata;
   - W3C Trace Context and OpenTelemetry;
   - IETF RateLimit headers.
2. **Nothing crashes.** A handler panic becomes a 500 and the process keeps
   serving. Requests have timeouts and size limits. Services shut down
   gracefully. `/health` is liveness and `/ready` checks each dependency. The
   version reported is the real release, never hard-coded.
3. **Everything ships to production.** Releases go `staging` → `main` through
   the merge queue. Deploys run on every push to `main`, with no variable
   gates. A missing secret fails loudly, never silently skips.
4. **Never close a conflicting PR.** Resolve the conflict and keep every
   feature.
5. **Think as a data scientist, a software engineer, a psychologist and an
   engagement designer at once.** Every record is linkable across systems by
   one canonical id: WorkOS `externalId` = Mongo entity id = ledger
   `owner_id`. An empty screen loses a user, so design for engagement and
   trust.
6. **Honesty.** Report what is measured, label design as design, and never
   overclaim.

**Dependencies.** Declare first-party packages (`@nyuchi/*`, `@bundu/*`,
`mzizi-*`) and direct dependencies as minimum versions (`>=x.y.z`) with no
upper bound; an old version blocks, the latest never does. Third-party
selectors known to break across majors, and runtime dependencies of
published packages, are capped within their major (`>=x <next-major`).

## What this repo is

`mzizi-dev/.github` is the org-level defaults repo for the `mzizi-dev` GitHub
organisation. It holds no application code: everything here is workflow YAML,
JSON rulesets, lint config and Markdown policy. `ORG_STANDARDS.md` is the
long-form record of what CI and governance actually enforce, so read the
relevant section before changing anything it describes.

## What it provides to the other mzizi-dev repos

| Path                                                                                                                                | Reaches other repos how                                                                                                                         |
| ----------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `profile/README.md`                                                                                                                 | Rendered as the org page at github.com/mzizi-dev                                                                                                |
| `.github/PULL_REQUEST_TEMPLATE.md`, `.github/ISSUE_TEMPLATE/`, `SECURITY.md`, `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `SUPPORT.md` | GitHub community-health fallback for any repo that lacks its own copy (only `mzizi-registry` ships its own today)                               |
| `.github/workflows/reusable-*.yml`                                                                                                  | Opt-in, via `on: workflow_call`                                                                                                                 |
| `.github/workflows/org-lint.yml`                                                                                                    | Named in the org ruleset's "Require workflows to pass" rule, so it runs on **every PR in every org repo**                                       |
| `workflow-templates/lint.yml` + `.properties.json`                                                                                  | Org starter workflow offered in each repo's Actions tab                                                                                         |
| `CODEOWNERS.example`, `dependabot.example.yml`                                                                                      | Templates to copy. CODEOWNERS and Dependabot have no org fallback; `.github/CODEOWNERS` and `.github/dependabot.yml` here govern this repo only |
| `github-rulesets/*.json`                                                                                                            | Proposals applied by hand with `gh api --input`; not applied automatically                                                                      |

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

- Work branches off `staging` and PRs land in `staging`. Each merge into
  `staging` is tagged the next patch (`staging-version.yml`); a release to
  `main` tags the next minor. Both use `nyuchi/.github`'s versioning
  policy, pinned by SHA. Never hand-edit a version to a major.
- Branch prefixes: `feat/`, `fix/`, `docs/`, or `claude/` for agent work.
- PR titles are Conventional Commits: lowercase, imperative subject, no
  trailing period (a subject starting with an all-caps word fails).
- Merges into `staging` are squash through the merge queue
  (`gh pr merge <n> --squash --auto`); the squash body keeps every commit
  message (`COMMIT_MESSAGES`). Updating a branch with its base is fine.
- `staging` is promoted to `main` with a **merge commit**, never squash:
  a merge commit keeps `main` at `staging` plus one merge, so the next
  release cannot conflict and nothing diverges.
- The default branch takes releases only: a PR into `main` must come from
  `staging`, a `release/*` branch, or the registry pin bot
  (`bot/registry-pin`). Everything else — features, fixes, Dependabot —
  targets `staging`.
- Anything that reaches `main` outside a release (hotfix, pin bot) is
  forward-ported to `staging` before the next release; the release check
  blocks the release until then (the `forward-ported` label).
- One writer per branch. Land with `gh pr merge <n> --auto`, which joins
  the merge queue; every workflow whose check is required must also
  trigger on `merge_group:`. Treat GitHub 5xx and "merge already in
  progress" as a wait, not a failure.
- Shared workflows are pinned estate-wide, every repo at the same commit:
  `nyuchi/.github/.github/workflows/<workflow>.yml@<40-hex sha>  # vX.Y.Z`
  (two spaces before `#`); Dependabot raises the pin after each release.
  Never `@main`, never a bare SHA without its tag comment.
- Advisory waivers are org-level only (`AUDIT_ALLOW_GHSAS` in
  `nyuchi/.github`), each with a reason and a review date; a repo cannot
  waive its own advisory.

## Checks that gate every pull request

- `org-lint.yml` in this repo is required on every PR in the org.
- The enterprise rulesets also run `nyuchi/.github`'s release version check,
  dependency review and Semgrep on every PR into `staging` and `main`.
- Every workflow whose check is required must also trigger on `merge_group:`.
- Never disable or weaken a check to get green; fix the cause or escalate.

- Never link `docs.mzizi.dev` in new prose (it does not resolve); see
  `ORG_STANDARDS.md#writing-a-readme` for branding facts READMEs get wrong.
