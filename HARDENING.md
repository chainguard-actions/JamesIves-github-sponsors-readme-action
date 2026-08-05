<!-- markdownlint-disable -->

# Hardening Report: JamesIves--github-sponsors-readme-action/v1.5.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **JamesIves--github-sponsors-readme-action/v1.5.4** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or branch names instead of pinned full-length SHA commit hashes, making them vulnerable to supply-chain attacks.

build.yml: actions/checkout@v4, actions/setup-node@v4 (×2), codecov/codecov-action@v4, actions/upload-artifact@v4
integration.yml: actions/checkout@v4, JamesIves/github-sponsors-readme-action@releases/v1 (×3)
label.yml: actions/checkout@v4, mauroalderete/action-assign-labels@v1
production.yml: actions/checkout@v4, actions/setup-node@v4
sponsors.yml: actions/checkout@v4, JamesIves/github-sponsors-readme-action@v1 (×2), JamesIves/github-pages-deploy-action@v4
version.yml: nowactions/update-majorver@v1.1.2, actions/checkout@v4, actions/setup-node@v4 (×2)

Locations:

- `.github/workflows/build.yml:16`
- `.github/workflows/build.yml:18`
- `.github/workflows/build.yml:31`
- `.github/workflows/build.yml:33`
- `.github/workflows/build.yml:47`
- `.github/workflows/integration.yml:15`
- `.github/workflows/integration.yml:18`
- `.github/workflows/integration.yml:24`
- `.github/workflows/integration.yml:33`
- `.github/workflows/label.yml:14`
- `.github/workflows/label.yml:18`
- `.github/workflows/production.yml:16`
- `.github/workflows/production.yml:18`
- `.github/workflows/sponsors.yml:12`
- `.github/workflows/sponsors.yml:20`
- `.github/workflows/sponsors.yml:33`
- `.github/workflows/sponsors.yml:44`
- `.github/workflows/version.yml:11`
- `.github/workflows/version.yml:22`
- `.github/workflows/version.yml:27`
- `.github/workflows/version.yml:50`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` keys on any job. Without explicit permissions, workflows run with the default (often overly broad) token permissions.

- build.yml
- integration.yml
- production.yml
- sponsors.yml
- version.yml

Locations:

- `.github/workflows/build.yml:1`
- `.github/workflows/integration.yml:1`
- `.github/workflows/production.yml:1`
- `.github/workflows/sponsors.yml:1`
- `.github/workflows/version.yml:1`

### script-injection (severity: high)

Rule (a): GitHub Actions expressions are interpolated directly inside `run:` shell command strings, bypassing shell quoting and enabling injection.

In production.yml, the step 'Commit and Push' interpolates `${{ secrets.GIT_CONFIG_EMAIL }}`, `${{ secrets.GIT_CONFIG_NAME }}`, and `${{ github.sha }}` directly inside the run block. While secrets.* values are not attacker-controlled, `github.sha` is a GitHub context value that flows through YAML template substitution before the shell sees it — any `${{ ... }}` in a run block is a script-injection finding per the check rules.

In version.yml, the step 'Authenticate with the GitHub Package Registry' interpolates `${{ secrets.GITHUB_TOKEN }}` directly inside the run block: `echo "//npm.pkg.github.com:_authToken=${{ secrets.GITHUB_TOKEN }}" > ~/.npmrc`.

Rule (b): In version.yml, the shell variable `$VERSION` (derived from the `GITHUB_REF` env var, which holds `${{ github.ref }}`) is used unquoted in `npm version $VERSION -m "Release $VERSION 📣"`, allowing shell metacharacter injection from a workflow-controllable value.

Locations:

- `.github/workflows/production.yml:36`
- `.github/workflows/version.yml:55`
- `.github/workflows/version.yml:47`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three finding types across 6 workflow files:

**unpinned-uses**: Pinned all action references to full SHA commit hashes with tag comments preserved:
- actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262
- actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020
- codecov/codecov-action@v4 → @b9fd7d16f6d7d1b5d2bec1a2887e65ceed900238
- actions/upload-artifact@v4 → @ea165f8d65b6e75b540449e92b4886f43607fa02
- JamesIves/github-sponsors-readme-action@releases/v1 and @v1 → @2fd9142e765f755780202122261dc85e78459405
- mauroalderete/action-assign-labels@v1 → @671a4ca2da0f900464c58b8b5540a1e07133e915
- JamesIves/github-pages-deploy-action@v4 → @d92aa235d04922e8f08b40ce78cc5442fcfbfa2f
- nowactions/update-majorver@v1.1.2 → @f2014bbbba95b635e990ce512c5653bd0f4753fb

**missing-permissions**: Added top-level permissions blocks to build.yml (contents: read), integration.yml (contents: read), production.yml (contents: write), sponsors.yml (contents: write), and version.yml (contents: write). label.yml already had permissions.

**script-injection**: In production.yml, moved GIT_CONFIG_EMAIL, GIT_CONFIG_NAME, and github.sha to step env block. In version.yml, moved GITHUB_TOKEN to step env block for the npmrc step, and quoted $VERSION in the npm version command to prevent shell metacharacter injection.

