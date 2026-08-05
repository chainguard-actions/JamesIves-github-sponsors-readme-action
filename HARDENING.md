<!-- markdownlint-disable -->

# Hardening Report: JamesIves--github-sponsors-readme-action/v1.5.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **JamesIves--github-sponsors-readme-action/v1.5.5** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files use tag-based or branch-based `uses:` references instead of pinned 40-character commit SHAs, making them vulnerable to supply-chain attacks if the referenced tag is moved or the action is compromised.

build.yml: actions/checkout@v4, actions/setup-node@v4, codecov/codecov-action@v5, actions/upload-artifact@v4
integration.yml: actions/checkout@v4, JamesIves/github-sponsors-readme-action@releases/v1 (×3)
label.yml: actions/checkout@v4, mauroalderete/action-assign-labels@v1
production.yml: actions/checkout@v4, actions/setup-node@v4
sponsors.yml: actions/checkout@v4, JamesIves/github-sponsors-readme-action@v1 (×2), JamesIves/github-pages-deploy-action@v4
version.yml: nowactions/update-majorver@v1.1.2, actions/checkout@v4, actions/setup-node@v4 (×2)

Locations:

- `.github/workflows/build.yml:16`
- `.github/workflows/integration.yml:14`
- `.github/workflows/label.yml:17`
- `.github/workflows/production.yml:15`
- `.github/workflows/sponsors.yml:12`
- `.github/workflows/version.yml:11`

### script-injection (severity: high)

Rule (a): In production.yml, the `github.sha` context is directly interpolated inside a `run:` shell command string: `git commit -m "Deploy Production Code for Commit ${{ github.sha }} 🚀"`. Any `${{ ... }}` expression inside a `run:` block flows through YAML template substitution before the shell processes it, bypassing shell quoting.

Rule (b): In version.yml, `${{ github.ref }}` is placed into the env var `GITHUB_REF`, which is then used unquoted in the shell: `VERSION=${GITHUB_REF#refs/tags/v}` and subsequently `npm version $VERSION` — `$VERSION` is unquoted, allowing shell metacharacter injection if the ref value contains special characters.

Locations:

- `.github/workflows/production.yml:38`
- `.github/workflows/version.yml:37`
- `.github/workflows/version.yml:39`

### missing-permissions (severity: medium)

Five workflow files have no top-level `permissions:` key and no job-level `permissions:` keys on any of their jobs. Without explicit permissions, workflows run with the default (potentially broad) token permissions, violating the principle of least privilege. Only label.yml has permissions defined.

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

1. unpinned-uses: Pinned all action references to full 40-char commit SHAs with tag comments preserved. Actions pinned: actions/checkout@v4, actions/setup-node@v4, codecov/codecov-action@v5, actions/upload-artifact@v4, JamesIves/github-sponsors-readme-action@releases/v1 and @v1, mauroalderete/action-assign-labels@v1, JamesIves/github-pages-deploy-action@v4, nowactions/update-majorver@v1.1.2.

2. script-injection: (a) production.yml: moved github.sha into COMMIT_SHA env var and also moved secrets into env vars; shell now uses $COMMIT_SHA. (b) version.yml: moved github.ref into GIT_REF env var; VERSION is now properly double-quoted as "$VERSION" in npm version call. Also moved GITHUB_TOKEN secret in npmrc step into env block.

3. missing-permissions: Added top-level permissions blocks to build.yml (contents: read), integration.yml (contents: read), production.yml (contents: write), sponsors.yml (contents: write), and version.yml (contents: write). label.yml already had permissions defined.

