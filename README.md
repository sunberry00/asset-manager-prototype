# asset-manager-prototype


[![Verify](https://github.com/sunberry00/asset-manager-prototype/actions/workflows/verify.yml/badge.svg)](https://github.com/sunberry00/asset-manager-prototype/actions/workflows/verify.yml)
[![CodeQL](https://github.com/sunberry00/asset-manager-prototype/actions/workflows/scan-codeql.yml/badge.svg)](https://github.com/sunberry00/asset-manager-prototype/actions/workflows/scan-codeql.yml)
[![OpenSSF Scorecard](https://api.securityscorecards.dev/projects/github.com/sunberry00/asset-manager-prototype/badge)](https://securityscorecards.dev/viewer/?uri=github.com/sunberry00/asset-manager-prototype)
[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](./LICENSE)
[![Release](https://img.shields.io/github/v/release/sunberry00/asset-manager-prototype)](https://github.com/sunberry00/asset-manager-prototype/releases)

# Asset manager for IMI

## Stack

bun workspaces monorepo, Vue 3 + TypeScript (Vite), Vitest with coverage gate, ESLint (flat) + Prettier, lefthook with shared hooks from aktin-github-scripts, tag-driven releases via git-cliff, and a hardened GitHub Actions pipeline (CI, CodeQL, Scorecard, dependency review, SBOM, build attestations).

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

To cut a release, check out the commit on `main` that you want to release, tag it `vX.Y.Z` and push the tag. Example for version 1.2.3:

```sh
git tag v1.2.3
git push origin v1.2.3
```

The tag starts `release.yml`, see [Workflows](.github/workflows/README.md#workflows).

## Repository settings

GitHub copies only the files of a template, not its settings or rulesets. Apply them once after `bun run init`, as described from section 4 on in the [new repository guide](https://github.com/aktin/aktin-github-scripts/blob/main/new-repository.md) of aktin-github-scripts.

The code owners in `.github/CODEOWNERS` review every pull request into `main`. The init script suggests `akomii`. Enter whoever is responsible for the reviews in this repository.

### Branch `dev` for releases

A repository that publishes releases uses the settings profile `release.json`. Features collect on `dev`, `dev` goes to `main` and `main` mirrors the released state. Do these steps before you apply the profile, because its rulesets block direct pushes afterwards. A repository without releases uses `no-release.json` and skips this section.

1. Create `dev` from `main` and push it:
   ```sh
   git switch -c dev main
   git push -u origin dev
   ```
2. In `.github/dependabot.yml` add `target-branch: dev` to both entries. Example for the first entry:
   ```yaml
   - package-ecosystem: bun
     directory: /
     target-branch: dev
     schedule:
       interval: weekly
   ```

### Private repository

A private repository needs two things that cost extra:

| Needs                   | For                               | Without it                                                                                                |
| ----------------------- | --------------------------------- | --------------------------------------------------------------------------------------------------------- |
| GitHub Code Security    | CodeQL and the dependency review  | `review / review` fails and CodeQL delivers no result, so the ruleset `default` blocks every pull request |
| GitHub Enterprise Cloud | The attestations in `release.yml` | The job `build` fails and no release is published                                                         |

To release without attestations, add the input `attest: false` to the job `build` in `.github/workflows/release.yml`. The build then skips the SBOM and the attestations, and the release holds only the bundle:

```yaml
build:
  needs: verify
  permissions:
    contents: read
    id-token: write
    attestations: write
  uses: aktin/aktin-github-scripts/.github/workflows/build-web.yml@main
  with:
    attest: false
```

## How the infra works

| Component                  | What it does                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| bun monorepo               | `apps/*` + `packages/*`, one lockfile, shared tooling at the root                                                                                                                                                                                                                                                                                                                                                                                |
| ESLint (flat) + Prettier   | ESLint owns correctness, Prettier owns formatting                                                                                                                                                                                                                                                                                                                                                                                                |
| Vitest + coverage          | Unit tests with an 80% line gate (`thresholds`)                                                                                                                                                                                                                                                                                                                                                                                                  |
| Pinned bun                 | Runtime version fixed in `.bun-version`                                                                                                                                                                                                                                                                                                                                                                                                          |
| lefthook                   | All hooks come from [aktin-github-scripts](https://github.com/aktin/aktin-github-scripts#git-hooks), see `lefthook.yml`. pre-commit formats and lints staged files, scans for secrets with gitleaks, lints staged workflows, Dockerfiles and shell scripts and rejects data files and large files, commit-msg enforces Conventional Commits and rejects oversized commits, pre-push runs `bun run typecheck`, `bun run test` and `bun run build` |
| GitHub workflows and rules | Checks on pull requests, CodeQL, dependency review, Scorecard, releases, Dependabot and the rulesets on `main` and release tags. Details in [.github/workflows/README.md](.github/workflows/README.md)                                                                                                                                                                                                                                           |

## Project structure

```
apps/web/          Vue 3 + Vite app (the default workspace)
packages/          shared packages (add as needed)
.github/workflows/ verify, CodeQL, dependency review, Scorecard, release, described in the README there
SECURITY.md        security policy
scripts/init.ts    one-time template initializer (removed after first run)
```

## Adding a backend or shared package

Create `apps/api` or `packages/shared` with its own `package.json`, extend `tsconfig.base.json`, and the root workspace globs pick it up. Root scripts (`build`, `test`, `typecheck`) fan out to every workspace.

## License

Apache-2.0. See [LICENSE](./LICENSE) and [NOTICE](./NOTICE).
