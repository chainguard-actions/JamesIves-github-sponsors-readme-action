<!-- markdownlint-disable -->

# Hardening Report: JamesIves--github-sponsors-readme-action/v1.5.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **JamesIves--github-sponsors-readme-action/v1.5.3** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files use tag-based or branch-based `uses:` references instead of pinned full 40-character SHA commit hashes, making them vulnerable to supply-chain attacks if the referenced tag is moved or the action is compromised.

build.yml: actions/checkout@v4, actions/setup-node@v4 (×2), codecov/codecov-action@v4, actions/upload-artifact@v4
integration.yml: actions/checkout@v4, JamesIves/github-sponsors-readme-action@releases/v1 (×3)
label.yml: actions/checkout@v4, mauroalderete/action-assign-labels@v1
production.yml: actions/checkout@v4, actions/setup-node@v4
sponsors.yml: actions/checkout@v4, JamesIves/github-sponsors-readme-action@v1 (×2), JamesIves/github-pages-deploy-action@v4
version.yml: nowactions/update-majorver@v1.1.2, actions/checkout@v4, actions/setup-node@v4 (×2)

Locations:

- `.github/workflows/build.yml:16`
- `.github/workflows/build.yml:19`
- `.github/workflows/build.yml:34`
- `.github/workflows/build.yml:47`
- `.github/workflows/build.yml:55`
- `.github/workflows/integration.yml:15`
- `.github/workflows/integration.yml:20`
- `.github/workflows/integration.yml:29`
- `.github/workflows/integration.yml:38`
- `.github/workflows/label.yml:17`
- `.github/workflows/label.yml:22`
- `.github/workflows/production.yml:17`
- `.github/workflows/production.yml:19`
- `.github/workflows/sponsors.yml:13`
- `.github/workflows/sponsors.yml:21`
- `.github/workflows/sponsors.yml:38`
- `.github/workflows/sponsors.yml:52`
- `.github/workflows/version.yml:12`
- `.github/workflows/version.yml:26`
- `.github/workflows/version.yml:31`
- `.github/workflows/version.yml:67`

### script-injection (severity: high)

GitHub Actions expressions (`${{ ... }}`) are interpolated directly inside `run:` shell command strings, violating rule (a). This causes the expression value to be substituted into the shell script before the shell parses it, which can allow injection of shell metacharacters.

- production.yml: `git config user.email "${{ secrets.GIT_CONFIG_EMAIL }}"` and `git config user.name "${{ secrets.GIT_CONFIG_NAME }}"` and `git commit -m "Deploy Production Code for Commit ${{ github.sha }} 🚀"` — all three expressions are directly interpolated into the run: shell script.
- version.yml: `echo "//npm.pkg.github.com:_authToken=${{ secrets.GITHUB_TOKEN }}" > ~/.npmrc` — the token expression is directly interpolated into the shell command string.

All values should be moved to `env:` variables and referenced as `"$ENV_VAR"` in the shell script.

Locations:

- `.github/workflows/production.yml:43`
- `.github/workflows/production.yml:44`
- `.github/workflows/production.yml:46`
- `.github/workflows/version.yml:62`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, the GITHUB_TOKEN is granted default (often write) permissions, violating the principle of least privilege.

- build.yml: no permissions defined at top level or in any job
- integration.yml: no permissions defined at top level or in any job
- production.yml: no permissions defined at top level or in any job
- sponsors.yml: no permissions defined at top level or in any job
- version.yml: no permissions defined at top level or in any job

Locations:

- `.github/workflows/build.yml:1`
- `.github/workflows/integration.yml:1`
- `.github/workflows/production.yml:1`
- `.github/workflows/sponsors.yml:1`
- `.github/workflows/version.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings across 6 workflow files:

**unpinned-uses**: Pinned all action references to full SHA hashes:
- actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262
- actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020
- codecov/codecov-action@v4 → @b9fd7d16f6d7d1b5d2bec1a2887e65ceed900238
- actions/upload-artifact@v4 → @ea165f8d65b6e75b540449e92b4886f43607fa02
- JamesIves/github-sponsors-readme-action@releases/v1 and @v1 → @2fd9142e765f755780202122261dc85e78459405
- mauroalderete/action-assign-labels@v1 → @671a4ca2da0f900464c58b8b5540a1e07133e915
- JamesIves/github-pages-deploy-action@v4 → @d92aa235d04922e8f08b40ce78cc5442fcfbfa2f
- nowactions/update-majorver@v1.1.2 → @f2014bbbba95b635e990ce512c5653bd0f4753fb

**script-injection**: Moved all ${{ }} expressions from run: scripts to env: blocks:
- production.yml: GIT_CONFIG_EMAIL, GIT_CONFIG_NAME, GITHUB_SHA
- version.yml: GITHUB_TOKEN for the npmrc authentication step

**missing-permissions**: Added top-level permissions blocks to build.yml (contents: read), integration.yml (contents: read), production.yml (contents: write), sponsors.yml (contents: write), and version.yml (contents: write, packages: write). label.yml already had permissions defined.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed the script injection vulnerability in hardened/action/.github/workflows/version.yml at line 56. Changed `npm version $VERSION` to `npm version "$VERSION"` to properly quote the shell variable expansion, preventing potential shell metacharacter injection if the tag name contains special characters. The `${{ github.ref }}` expression was already correctly placed in the step's env block as GITHUB_REF.

