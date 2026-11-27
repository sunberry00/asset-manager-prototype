# CI/CD and repository rules

What runs on GitHub for this repository and which rules protect `main`. The rules apply once they are set up as described under [Repository settings](../../README.md#repository-settings) in the root README, which also covers the git hooks that run on your machine.

## On a pull request

These checks appear on a pull request into `main`.

| Check                    | Workflow                                                                                                   | Required to merge         |
| ------------------------ | ---------------------------------------------------------------------------------------------------------- | ------------------------- |
| `verify / verify`        | `verify.yml`, passes only when `verify / bun` and `verify / base / verify` passed                          | yes                       |
| `verify / bun`           | `verify.yml`, runs format check, lint, typecheck, tests and build                                          | through `verify / verify` |
| `verify / base / verify` | `verify.yml`, lints Dockerfiles, shell scripts and workflow files                                          | through `verify / verify` |
| `review / review`        | `scan-dependencies.yml`, fails on new dependencies with a known vulnerability of severity high or critical | no                        |
| `analyze / analyze`      | `scan-codeql.yml`, CodeQL scan of the JavaScript and TypeScript code and the workflow files                | no                        |
| `CodeQL`                 | no workflow file, GitHub code scanning posts it with the result of the CodeQL scan                         | no                        |

## Workflows

The workflow files only hold the triggers. Their jobs live as reusable workflows in [aktin-github-scripts](https://github.com/aktin/aktin-github-scripts/tree/main/.github/workflows) and are called at `@main`. The actions they use are pinned to commit SHAs there.

What each workflow does and when it runs. A pull request means one into `main`. Schedules are in UTC and run on `main`:

| Workflow                | Pull request | Push to `main` | Schedule      | Other | What happens                                                                                                                                                                                                                                                                                                                                                                                                    |
| ----------------------- | ------------ | -------------- | ------------- | ----- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `verify.yml`            | yes          | yes            |               |       | Starts the shared `verify-web.yml`. It checks the Dockerfiles with hadolint, the shell scripts with shellcheck and the workflow files with actionlint, installs the dependencies from `bun.lock` and runs the scripts `format:check`, `lint`, `typecheck`, `test` and `build`. The run fails when hadolint reports an error, shellcheck a warning or worse, actionlint any finding, or one of the scripts fails |
| `scan-dependencies.yml` | yes          |                |               |       | Compares the dependencies before and after the pull request and fails when a newly added one has a known vulnerability of severity high or critical. On failure it writes the findings as a comment into the pull request                                                                                                                                                                                       |
| `scan-codeql.yml`       | yes          | yes            | Mondays 03:00 |       | Scans the JavaScript and TypeScript code with CodeQL and reports the findings in the Security tab. It also scans the workflow files. Shell scripts are not scanned                                                                                                                                                                                                                                              |
| `scan-scorecard.yml`    |              | yes            | Mondays 04:00 |       | Rates the security practices of the repository, for example branch rules, pinned actions, token permissions and code review, publishes the score to OpenSSF and to the Security tab and feeds the badge in the root README. It changes nothing                                                                                                                                                                  |

In `verify.yml` and `scan-codeql.yml` a newer run for the same pull request or branch cancels the one that is still running.

Dependabot is configured in `.github/dependabot.yml`. Once a week it opens one grouped pull request per ecosystem, one for the bun dependencies and one for the GitHub Actions referenced in the workflow files. These pull requests go to `main`. Dependabot alerts and security updates are switched on in the repository settings and report known vulnerabilities in dependencies that are already in use. Security updates always open their pull requests against `main`. GitHub does not support them for bun yet, so these pull requests fail `verify / verify`.

## Gates on `main`

Branch ruleset `default` on `main`:

| Rule                     | Meaning                                                                              |
| ------------------------ | ------------------------------------------------------------------------------------ |
| Pull request required    | No direct pushes. Squash merge only. No approval needed, all review threads resolved |
| Required checks          | `verify / verify` must pass                                                          |
| Signed commits           | Every commit on `main` is signed. Squash merge commits are signed by GitHub          |
| No force push, no delete | History on `main` is append-only                                                     |

The bypass list of the ruleset names who can skip its rules. No app or bot is on it. Other branches and tags have no rules.
