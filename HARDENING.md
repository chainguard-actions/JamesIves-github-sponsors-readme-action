<!-- markdownlint-disable -->

# Hardening Report: JamesIves--github-sponsors-readme-action/v1.5.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **JamesIves--github-sponsors-readme-action/v1.5.6** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): ${{ ... }} expressions are directly interpolated inside run: shell command strings. In production.yml, the 'Commit and Push' step interpolates ${{ secrets.GIT_CONFIG_EMAIL }}, ${{ secrets.GIT_CONFIG_NAME }}, and ${{ github.sha }} directly into shell commands. In version.yml, the 'Authenticate with the GitHub Package Registry' step interpolates ${{ secrets.GITHUB_TOKEN }} directly into an echo command: `echo "//npm.pkg.github.com:_authToken=${{ secrets.GITHUB_TOKEN }}" > ~/.npmrc`. Any ${{ ... }} expression inside a run: block is a script-injection risk because YAML template substitution happens before the shell ever sees the value.

Locations:

- `.github/workflows/production.yml:38`
- `.github/workflows/production.yml:39`
- `.github/workflows/production.yml:41`
- `.github/workflows/version.yml:55`

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags instead of immutable 40-character SHA digests, making them vulnerable to supply-chain attacks if the tag is moved. Failing references include: build.yml — actions/checkout@v4, actions/setup-node@v4, codecov/codecov-action@v5, actions/upload-artifact@v4; integration.yml — actions/checkout@v4, JamesIves/github-sponsors-readme-action@releases/v1 (×3); label.yml — actions/checkout@v4, mauroalderete/action-assign-labels@v1; production.yml — actions/checkout@v4, actions/setup-node@v4; sponsors.yml — actions/checkout@v4, JamesIves/github-sponsors-readme-action@v1 (×2), JamesIves/github-pages-deploy-action@v4; version.yml — nowactions/update-majorver@v1.1.2, actions/checkout@v4, actions/setup-node@v4 (×2).

Locations:

- `.github/workflows/build.yml:16`
- `.github/workflows/integration.yml:15`
- `.github/workflows/label.yml:17`
- `.github/workflows/production.yml:15`
- `.github/workflows/sponsors.yml:12`
- `.github/workflows/version.yml:11`

### missing-permissions (severity: medium)

Five workflow files have no top-level permissions: block and no job-level permissions: blocks on any of their jobs. Without explicit permissions, the GITHUB_TOKEN is granted its default (often broad) permissions, violating the principle of least privilege. The affected files are build.yml, integration.yml, production.yml, sponsors.yml, and version.yml. (label.yml correctly declares top-level permissions.)

Locations:

- `.github/workflows/build.yml:1`
- `.github/workflows/integration.yml:1`
- `.github/workflows/production.yml:1`
- `.github/workflows/sponsors.yml:1`
- `.github/workflows/version.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across 6 workflow files:

1. script-injection: In production.yml, moved ${{ secrets.GIT_CONFIG_EMAIL }}, ${{ secrets.GIT_CONFIG_NAME }}, and ${{ github.sha }} from run: shell strings into env: blocks. In version.yml, moved ${{ secrets.GITHUB_TOKEN }} from the echo command into an env: block, referencing it as ${GITHUB_TOKEN} in the shell.

2. unpinned-uses: Pinned all 9 distinct action references to their full 40-character SHA digests across build.yml, integration.yml, label.yml, production.yml, sponsors.yml, and version.yml. Original tag names preserved as inline comments.

3. missing-permissions: Added top-level permissions blocks to build.yml (contents: read), integration.yml (contents: read), production.yml (contents: write for git push), sponsors.yml (contents: write for GitHub Pages deploy), and version.yml (contents: write + packages: write for publishing). label.yml already had correct permissions and was only updated to pin action SHAs.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed the unquoted `$VERSION` variable in the 'Set version to match the tag' step in `.github/workflows/version.yml`. Changed `npm version $VERSION -m "Release $VERSION 📣"` to `npm version "$VERSION" -m "Release $VERSION 📣"`. The first positional argument to `npm version` was unquoted, allowing shell metacharacter injection if the tag ref contained special characters. The fix adds double quotes around the first `$VERSION` to prevent this.

