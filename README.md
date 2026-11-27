# asset-manager-prototype

[![Verify](https://github.com/sunberry00/asset-manager-prototype/actions/workflows/verify.yml/badge.svg)](https://github.com/sunberry00/asset-manager-prototype/actions/workflows/verify.yml)
[![CodeQL](https://github.com/sunberry00/asset-manager-prototype/actions/workflows/scan-codeql.yml/badge.svg)](https://github.com/sunberry00/asset-manager-prototype/actions/workflows/scan-codeql.yml)
[![OpenSSF Scorecard](https://api.securityscorecards.dev/projects/github.com/sunberry00/asset-manager-prototype/badge)](https://securityscorecards.dev/viewer/?uri=github.com/sunberry00/asset-manager-prototype)
[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](./LICENSE)

# Asset manager for IMI

## Stack

bun workspaces monorepo, Vue 3 + TypeScript (Vite), Vitest with coverage gate, ESLint (flat) + Prettier, lefthook with shared hooks from aktin-github-scripts, and a hardened GitHub Actions pipeline (CI, CodeQL, Scorecard, dependency review).

## Getting started

Prerequisites: [bun](https://bun.sh) on PATH (the version is pinned in `.bun-version`). The shared git hooks need `python3`, `gitleaks`, `actionlint`, `hadolint` and `shellcheck` on PATH. A missing tool fails the commit. actionlint, hadolint and shellcheck are only called when a workflow file, Dockerfile or shell script is staged.

```sh
bun install        # deps + git hooks
bun run dev        # start the web app (apps/web)
bun run test       # unit tests with coverage
bun run typecheck  # vue-tsc
bun run lint       # eslint
bun run format     # prettier
bun run build      # production build
```

The coverage report lands in `apps/web/coverage/index.html`.

Commits must follow [Conventional Commits](https://www.conventionalcommits.org), enforced by the commit-msg hook:

```
<type>(<optional scope>): <subject>
```

Allowed types are `feat fix docs style refactor perf test chore`, the subject starts lowercase and has no trailing period, every line of the message stays at most 100 characters.

## Repository settings

The repository was created with `new-repo.sh` from [aktin-github-scripts](https://github.com/aktin/aktin-github-scripts) which also applied the settings profile `no-release.json`. To apply it again, run from the root of aktin-github-scripts:

```sh
repo-settings/apply.sh sunberry00/asset-manager-prototype repo-settings/no-release.json
```

GitHub asks the code owners in `.github/CODEOWNERS` for a review of every pull request into `main`. The merge does not wait for it.

## How the infra works

| Component                  | What it does                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| bun monorepo               | `apps/*` + `packages/*`, one lockfile, shared tooling at the root                                                                                                                                                                                                                                                                                                                                                                                |
| ESLint (flat) + Prettier   | ESLint owns correctness, Prettier owns formatting                                                                                                                                                                                                                                                                                                                                                                                                |
| Vitest + coverage          | Unit tests with an 80% line gate (`thresholds`)                                                                                                                                                                                                                                                                                                                                                                                                  |
| Pinned bun                 | Runtime version fixed in `.bun-version`                                                                                                                                                                                                                                                                                                                                                                                                          |
| lefthook                   | All hooks come from [aktin-github-scripts](https://github.com/aktin/aktin-github-scripts#git-hooks), see `lefthook.yml`. pre-commit formats and lints staged files, scans for secrets with gitleaks, lints staged workflows, Dockerfiles and shell scripts and rejects data files and large files, commit-msg enforces Conventional Commits and rejects oversized commits, pre-push runs `bun run typecheck`, `bun run test` and `bun run build` |
| GitHub workflows and rules | Checks on pull requests, CodeQL, dependency review, Scorecard, Dependabot and the ruleset on `main`. Details in [.github/workflows/README.md](.github/workflows/README.md)                                                                                                                                                                                                                                                                       |

## Project structure

```
apps/web/          Vue 3 + Vite app (the default workspace)
packages/          shared packages (add as needed)
.github/workflows/ verify, CodeQL, dependency review, Scorecard, described in the README there
```

## Adding a backend or shared package

Create `apps/api` or `packages/shared` with its own `package.json`, extend `tsconfig.base.json`, and the root workspace globs pick it up. Root scripts (`build`, `test`, `typecheck`) fan out to every workspace.

## License

Apache-2.0. See [LICENSE](./LICENSE) and [NOTICE](./NOTICE).
