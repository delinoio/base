# Repository Workflow Contract

## Scope

The base repository contains shared instructions, skills, documentation templates and formatting infrastructure. It has no runtime projects, native build workspace, generated protocol bindings, deployment targets or release credentials.

## Installation

Use Node.js 24 from `.nvmrc` and pnpm 10.26.2 from `package.json`. Run `pnpm install --frozen-lockfile`. The prepare script installs Lefthook through `scripts/install-git-hooks.sh`. Archives without Git metadata skip hook installation. Linked worktrees may use their shared Git hooks directory; unrelated custom hook paths remain protected.

## Formatting

Root `pnpm format` writes fixes and `pnpm format:check` checks without rewriting. Both use pinned Rstack and `rstack.config.ts`. Default local selection compares the merge base of `origin/main` and HEAD, including working changes and untracked files. Explicit `--base <sha> --head <sha>` arguments select that committed comparison. Local `--all` selects existing tracked files. Before the first commit, use `--staged`.

Lefthook runs `node scripts/ci/format-changed.mjs --staged --write` with `stage_fixed: true`. It preserves unstaged edits, supports initial commits and rejects parser failures. Select files with NUL-separated Git paths, skip deletions and symlinks, and preserve imported skills, generated files and lockfiles through shared exclusions.

CI requires an explicit comparison and rejects local `--staged` and `--all` modes. An unset, empty or case-insensitive `false` CI value means local execution; every other nonempty value enables CI restrictions.

## Validation

Run `pnpm test` for formatter, hook and change-planning fixtures. CI plans affected fixtures and `pnpm format:check --base <sha> --head <sha>` on Ubuntu for pull requests, main pushes and manual dispatch. PR comparisons use the pull request base and head. Push comparisons use the prior and current commits. Manual comparisons use the merge base with `origin/main`; an optional `base` input selects another existing revision. A root push has no previous commit, so CI creates a local empty-tree comparison commit without pushing it. No full manual mode or macOS runners are configured.

The `changes` job resolves that comparison and runs `scripts/ci/plan.mjs` without installing dependencies. It publishes `base`, `head`, `format`, `fixtures` and `quality` outputs. Formatting uses the existing formatter's changed-file selection; Rstack still owns exclusions. Fixtures run for changes, deletions or renames affecting `scripts/ci/`, `scripts/install-git-hooks.sh`, `.github/workflows/CI.yml`, `.nvmrc`, `package.json`, `pnpm-lock.yaml`, `pnpm-workspace.yaml`, `rstack.config.ts` or `lefthook.yml`. Documentation-only changes select formatting without fixtures. The `quality` job runs when either check is selected and passes the exact comparison to formatting. Empty comparisons or changes with no validation inputs skip `quality`. The planner records the checkout revision, comparison and selections in logs and the job summary. New shared validation dependencies must update the planner inputs.

Each CI run records the validation revision, comparison revisions, commands and results in its logs and job summary. Runtime and platform acceptance must be added with the owning project. Passing shared fixtures does not establish runtime or release acceptance.

The `ci-result` job aggregates validation through `needs`, including `changes` and `quality`. It runs with `always()` so dependency failures do not skip the aggregate check. Failed or cancelled dependencies fail `ci-result`. The `changes` job must succeed; a skipped `quality` job passes only when the planner selected no validation. Successful dependencies and other skipped dependencies pass. Every new PR validation job must be included in `ci-result.needs`. The aggregate job runs on Ubuntu for the same PR, main-push and manual-dispatch events.

## Assets and licensing

Every repository-owned `dist` directory is ignored generated output and must be removed after validation. Assets at least 512 KiB require exact-path Git LFS attributes and hydration before use. There are no LFS assets in this base.

## Dependency updates

Renovate extends `github>delinoio/renovate-config`. Ordinary updates and lockfile maintenance use Monday 00:00–04:00 in `Asia/Seoul`; vulnerability updates retain the inherited immediate security policy. Follow the dependency security contract and retain unresolved findings in GitHub or CI records.
