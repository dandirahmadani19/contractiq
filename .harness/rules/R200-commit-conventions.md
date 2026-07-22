# R200 — Commit, Branch, PR, and Release Conventions

Rules governing git history, PR workflow, and release automation.
Enforcement is mostly automated (commitlint, Lefthook, GitHub
Actions); the rest is reviewer discipline.

## Scope

All commits to any branch of `github.com/<user>/contractiq`. Does
not apply to `contractiq-learning-docs` (personal notes, more
lenient).

---

## R200.1 — Conventional Commits Format

**MUST**: every commit subject matches this grammar:

```
<type>(<scope>): <subject>

[optional body]

[optional footer(s)]
```

Where:
- `<type>` is one of the types in R200.2.
- `<scope>` is one of the scopes in R200.3 (parentheses required
  even when empty for global changes? — no, empty scope omits the
  parens; see examples).
- `<subject>` follows R200.4.

**MUST NOT**: use free-form subject lines. Enforced by commitlint
via Lefthook `commit-msg` hook.

### Examples (valid)
```
feat(api): add extract_clauses tool endpoint
fix(web): correct focus trap in Dialog close on Escape
ai(llm): tune risk assessment prompt for liability clauses
db(api): add contracts.jurisdiction column
docs: fix typo in ARCHITECTURE.md
chore: bump pnpm to 9.15
```

### Examples (invalid)
```
Add feature                                # ❌ no type
feat: add stuff                            # ❌ vague subject
FEAT(api): add tool                        # ❌ type must be lowercase
fix(api) missing colon                     # ❌ missing colon
feat(api): Added extraction tool.          # ❌ past tense + period
```

## R200.2 — Type Whitelist

**MUST**: use exactly one of these types. Extending the list requires
a new ADR that supersedes ADR-021.

| Type       | When                                                      |
| ---------- | --------------------------------------------------------- |
| `feat`     | User-visible new feature or capability                    |
| `fix`      | User-visible bug fix                                      |
| `refactor` | Restructure code, no behavior change                      |
| `perf`     | Performance improvement, no behavior change               |
| `test`     | Add or modify tests only                                  |
| `docs`     | Documentation only (`.md`, `apps/docs`, code comments)    |
| `chore`    | Tooling, dependency bumps, config                         |
| `ci`       | GitHub Actions or CI-related config                       |
| `build`    | Build system, bundler, Turbo config                       |
| `ai`       | Prompt content, tool schemas, LLM adapter, eval datasets  |
| `db`       | Prisma schema, migrations, DB seed                        |
| `sec`      | Security-related change (auth, validation, dep with CVE)  |

**Choosing between types when overlapping**:

- Prompt change that ships a new user feature → `feat` (feature
  is user-visible; prompt is implementation detail).
- Prompt change to improve accuracy without new capability → `ai`.
- DB migration required by a new feature → **two commits**: `db`
  first, `feat` second.
- Dependency bump that patches a CVE → `sec`, not `chore`.
- Test-only PR that doesn't ship a feature → `test`.
- Docs on a new feature that will ship in the same PR → `feat`
  (docs are part of Definition of Done per MVP.md).

## R200.3 — Scope Whitelist

**MUST**: use one of these scopes, or omit scope for repo-global
changes.

| Scope       | Covers                                                     |
| ----------- | ---------------------------------------------------------- |
| `api`       | `apps/api`                                                 |
| `web`       | `apps/web`                                                 |
| `docs`      | `apps/docs` (Nextra site content, not `.md` in repo root)  |
| `shared`    | `packages/shared`                                          |
| `ui`        | `packages/ui`                                              |
| `db`        | `packages/db` (schema, client, migrations)                 |
| `llm`       | `packages/llm`                                             |
| `evals`     | `packages/evals`                                           |
| `config`    | `packages/config`                                          |
| `harness`   | `.harness/**`, `CLAUDE.md`                                 |
| `ci`        | `.github/**`                                               |
| `tooling`   | `tooling/**`, root scripts                                 |
| `deps`      | Dependency updates that touch multiple packages            |

**Repo-global changes** (omit scope):
- Changes to root `README.md`, `CONTRIBUTING.md`, `SECURITY.md`.
- Changes to `DECISIONS.md`, `ARCHITECTURE.md`, `MVP.md`, `DESIGN.md`.
- Repo-wide `.gitignore`, `.editorconfig`, `.gitattributes`.
- Multi-package refactors that don't cleanly attribute to one scope.

### Examples
```
docs: add ARCHITECTURE.md                  # global, no scope
feat(api): add /contracts POST endpoint    # scoped to api
refactor(ui): split Dialog into subcomps   # scoped to ui
chore(deps): bump zod to 3.24              # cross-package deps
```

## R200.4 — Subject Line Rules

**MUST**:
- Lowercase first character (except proper nouns, code identifiers).
- Imperative mood ("add", not "added" or "adds").
- No trailing period.
- ≤ 72 characters total including type, scope, colon.

**SHOULD**:
- Describe the *change*, not the file. Prefer
  "add extract_clauses tool" over "modify extract-clauses.ts".
- Be specific enough that someone reading `git log --oneline`
  understands what shipped.

### Violation
```
feat(api): Updated the file that handles contract extraction to add support for more clause types.
# ❌ >72 char, past tense, capital, vague file-oriented phrasing
```

### Correct
```
feat(api): support liability clauses in extract_clauses tool
```

## R200.5 — Body Rules

**MUST**:
- Blank line between subject and body.
- Wrap body at 100 characters.
- Explain *why* the change was made when it isn't obvious from
  the subject; describe *what* only if the code diff doesn't
  self-explain.

**MAY**: omit body entirely for trivial commits (single-line
subject is sufficient).

**MUST**: include body when the commit:
- Introduces a workaround (explain why the direct fix isn't taken).
- Reverts a prior commit (explain the failure mode observed).
- Touches security-relevant code (`sec`, or auth-related `feat`).
- Adjusts a threshold (rate limit, timeout, retry count) — record
  the reasoning.

### Example
```
perf(api): batch chunk embeddings in groups of 32

Single-chunk embed calls exceeded 400ms p95 due to per-call TLS
handshake. Batching to 32 brings p95 to 90ms on the same input
volume. 32 chosen because Gemini rejects >64 in a single call
and 32 is a comfortable margin.

Verified with: pnpm --filter @contractiq/llm test:perf:embed
```

## R200.6 — Footer Rules

Footers are structured trailers. Zero or more per commit. One
footer per line, blank line above the block.

**Recognized trailers**:

| Trailer               | Purpose                                     |
| --------------------- | ------------------------------------------- |
| `Refs: #123`          | Non-closing reference to an issue           |
| `Closes: #123`        | GitHub keyword — closes issue on merge      |
| `Fixes: #123`         | GitHub keyword — closes issue on merge      |
| `BREAKING CHANGE:`    | Semantic-versioning major bump signal       |
| `Co-authored-by:`     | Multi-author credit                         |
| `Related-ADR: ADR-###`| Point to the decision that authorized this  |
| `Related-Rule: R###`  | Point to the rule this commit implements    |
| `Related-Task: T-###` | Point to the MVP.md task closed by commit   |

**MUST**: include `BREAKING CHANGE:` when the diff changes a public
contract in a way that requires downstream code changes. See R200.7.

### Example
```
feat(llm): add rerank capability to LLMProvider interface

Adds optional `rerank` method to enable ADR-013's contextual
retrieval pipeline. Providers without native rerank return
`undefined` and callers fall back to bge-reranker-v2-m3.

Related-ADR: ADR-013
Related-Task: T-036
```

## R200.7 — Breaking Changes

**MUST**: mark breaking changes explicitly in the footer.

A change is **breaking** when any of the following hold:

- Public function signature changes (removed param, changed types,
  return type differs).
- Environment variable removed or renamed.
- Public HTTP API contract changed (removed endpoint, changed
  status code, changed response shape).
- DB migration is not backward-compatible with the prior version.
- A workspace-level configuration file requires manual edit.

**MUST NOT**: mark internal refactors as breaking, even if many
files change.

### Example
```
feat(api)!: rename /contracts/:id/analysis to /contracts/:id/reports

BREAKING CHANGE: The GET /contracts/:id/analysis endpoint is removed;
use GET /contracts/:id/reports instead. Web client updated in this
same commit; external API consumers must migrate.

Related-Task: T-052
```

Note the `!` after scope — additional visual signal.

## R200.8 — Atomic Commits

**MUST**: each commit represents one logical change.

- If a commit touches `packages/db` (migration) *and* `apps/api`
  (usage of new column) *and* `apps/web` (UI for it), that is
  usually one logical feature — one commit is fine.
- If a commit fixes bug A and refactors unrelated code B, that is
  two logical changes — split into two commits.

**SHOULD**: prefer many small commits during development, then
squash irrelevant WIP before pushing (see R200.9).

**MUST NOT**: bundle unrelated changes to save a PR (splitting
into two PRs is cheap; bad history is expensive).

## R200.9 — Local History Hygiene

**MAY**: use `git commit --amend` and interactive rebase (`git rebase -i`)
to clean up local history before pushing.

**MUST NOT**: rewrite history on `main` or any branch that has been
pushed and referenced by CI or another human. Force-push is
allowed only on your own feature branches before opening a PR.

**MUST**: run `git log --oneline` and read every commit subject
before pushing a feature branch. If any subject is vague or wrong,
fix it locally.

## R200.10 — Signed Commits

**MUST**: every commit signed with SSH signing. GitHub shows
"Verified" badge on signed commits.

Local setup (one-time, not committed):
```bash
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global gpg.format ssh
git config --global commit.gpgsign true
```

`allowedSignersFile` configured per repo pointing to
`.github/allowed_signers` (committed).

**Enforcement**: Fase 2 checkpoint verifies latest commit shows
Verified on GitHub. CI does not currently reject unsigned commits
(GitHub Actions can't enforce cheaply); reviewer check.

## R200.11 — Branch Naming

**MUST**: use the format `<type>/<short-description>`, where `<type>`
matches R200.2 whitelist.

**SHOULD**: `<short-description>` is 2–5 kebab-case words.

### Examples (valid)
```
feat/extract-clauses-tool
fix/dialog-focus-trap
refactor/tenant-context-plugin
docs/architecture-diagrams
```

### Examples (invalid)
```
Development                    # ❌ no type
feature/big-changes            # ❌ 'feature' isn't in whitelist
feat/T-039                     # ❌ task ID isn't descriptive
feat/extract_clauses_tool      # ❌ underscore, not kebab
```

**MUST NOT**:
- Use spaces or special characters.
- Reuse a branch name after it's been merged and deleted.

## R200.12 — Branch Lifetime

**MUST**:
- Feature branches created from `main`.
- Feature branches merged back within ≤ 2 days of creation, or
  rebased on `main` to stay current.
- Delete branches after merge (GitHub auto-delete on merge is
  enabled).

**MUST NOT**:
- Long-lived integration branches (no `develop`).
- Branches from other branches (except when the parent is a very
  recent WIP branch you own).

Rationale: solo dev, trunk-based, keeps `main` always deployable
(ADR-022).

## R200.13 — PR Requirements

**MUST**: open a PR for every merge to `main`. No direct pushes.

Each PR MUST:
- Have a semantic title (Conventional Commits format — same rules
  as commit subject).
- Include a description with sections: **What**, **Why**,
  **How to verify**, **Rule citations**.
- Link the closing task (`Closes T-###`) if it closes an MVP task.
- Pass all 15 CI gates (ADR-020).

**SHOULD**:
- Small PRs (≤ 400 lines diff) merge faster and review better.
  Larger PRs need a body explaining the scope.
- Include screenshots for UI changes.
- Include LLM eval delta for `ai:` PRs.

### PR body template
```markdown
## What
<one-sentence summary>

## Why
<link to ADR / task / user need>

## How to verify
<exact commands the reviewer/self runs>

## Rule citations
<R###, R### applied or waived>

Closes T-###
```

Stored as `.github/PULL_REQUEST_TEMPLATE.md` — reference in the
repo, not repeated here.

## R200.14 — Self-Review Discipline

Because ContractIQ is solo-dev, PRs are self-reviewed. Discipline
substitutes for a second pair of eyes.

**MUST**: before merging your own PR:

1. Read the diff top-to-bottom in the GitHub UI, not just locally.
2. Read every commit subject and body — confirm they still describe
   what shipped after your rebase / amendments.
3. Verify CI is green (all 15 gates).
4. Run one manual test that touches the changed surface even if CI
   covers it.
5. Add a review comment on any line where you knowingly made a
   trade-off; future-you thanks present-you.

**MUST NOT**: merge immediately after opening the PR. Wait for CI
to complete even if you're confident.

## R200.15 — Merge Strategy

**MUST**: merge via **squash** by default.

**MAY**: merge via **rebase** when the branch's commit history is
already atomic and clean (rare but valuable — preserves fine-grained
history in `main`).

**MUST NOT**: merge via **create merge commit** — polluted history
with merge commits obscures rebase-based bisect.

Rationale: squash produces one commit per PR, mapping cleanly to
Changesets and the release changelog.

## R200.16 — Changesets Integration

**MUST**: for any PR that ships a change to a versioned package
(anything under `packages/*` that publishes semantic-versioned
outputs, and eventually `apps/*` with release cadence), include a
`.changeset/*.md` file describing the change.

Changeset file format:
```markdown
---
"@contractiq/llm": minor
"@contractiq/shared": patch
---

Add rerank capability to LLMProvider. Adapters must implement the
new optional method; missing implementations fall back to
bge-reranker-v2-m3 self-hosted (R500).
```

**MUST NOT**: include changesets for internal-only changes
(harness docs, root markdown, learning_docs).

CI enforces: any PR touching `packages/*` sources without a
`.changeset/*.md` is blocked with a comment linking to R200.16.

**Bump rules**:
- `major`: breaks public API (matches R200.7 breaking).
- `minor`: adds public API, backward-compatible.
- `patch`: bugfix, perf, internal refactor visible only via
  changed behavior.

## R200.17 — Release Cadence

**MUST**: releases produced by Changesets on merge to `main`.

The release workflow:

1. On merge to `main`, GitHub Action runs Changesets.
2. If unreleased changesets exist, Changesets opens (or updates) a
   "Version Packages" PR that bumps versions and updates
   `CHANGELOG.md` for each affected package.
3. When the "Version Packages" PR merges, another Action:
   - Tags the release (e.g., `@contractiq/llm@0.3.0`).
   - Publishes to npm (if we ever publish; currently private).
   - Deploys to production (Vercel, Fly.io) via their integrations.

**MUST NOT** hand-edit `CHANGELOG.md` or version fields in
`package.json` — Changesets owns them.

## R200.18 — Reverting

**MUST**: use `git revert <commit>` (not `git reset` on `main`).

Revert commits themselves follow R200.1–R200.7:

```
revert: "feat(api): add extract_clauses tool endpoint"

This reverts commit abc1234. Reason: tool schema needs redesign to
support nested clauses (see #45).

Related-Task: T-039
```

**MUST NOT** silently drop code — if a change is undesired after
merge, revert it in the open and file a follow-up.

## R200.19 — Fix-Up Commits

**MAY**: use `git commit --fixup=<sha>` during PR iteration. Before
merging, autosquash with `git rebase -i --autosquash origin/main`.

**MUST NOT** leave `fixup!` commits in `main` — squash strategy in
R200.15 hides them for squash merges, but rebase merges must
autosquash first.

## R200.20 — Commit Frequency

**SHOULD** commit at least once per meaningful step during development.

Signals a commit is warranted:
- A test that failed now passes.
- A refactor step compiles cleanly.
- A file moved or renamed.
- End of a work session.

Avoid the two extremes:
- Massive "wip" commits over multi-day work.
- Per-keystroke micro-commits that clutter history.

Aim for commits that would each be defensible in a `git log` review
as one logical unit.

## R200.21 — Lefthook Enforcement Summary

Lefthook (`lefthook.yml` at repo root) runs the following on
`commit-msg`:

- `commitlint`: R200.1–R200.7 grammar check.

And on `pre-commit`:

- `biome check --apply-unsafe`: format staged files.
- `tsc --noEmit` on affected packages via Turborepo.
- `pnpm test:affected` (fast subset).

And on `pre-push`:

- `tsc --noEmit` all packages (final gate before remote).
- Verify current branch matches R200.11 pattern (bash regex check).

Waivers per R100.22 apply here too: emergency bypass with
`git commit --no-verify` requires a follow-up note explaining
why.

## R200.22 — CI Enforcement Summary

| Rule           | Tool                          | Gate     |
| -------------- | ----------------------------- | -------- |
| R200.1–.7      | commitlint (Lefthook + CI)    | blocking |
| R200.10        | GitHub UI Verified badge      | reviewer |
| R200.11        | Lefthook `pre-push` regex     | blocking |
| R200.13        | GitHub PR title check action  | blocking |
| R200.13 (body) | Missing section = warn        | reviewer |
| R200.14        | Manual                        | self     |
| R200.15        | GitHub repo setting           | enforced |
| R200.16        | Changesets Action             | blocking |
| R200.17        | Changesets Action             | automated|
| R200.18–.20    | Reviewer                      | self     |
| R200.21        | Lefthook local hooks          | blocking |
