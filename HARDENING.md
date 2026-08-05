<!-- markdownlint-disable -->

# Hardening Report: JamesIves--github-sponsors-readme-action/v1.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **JamesIves--github-sponsors-readme-action/v1.6.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable version tags instead of pinned 40-character SHA digests, making them vulnerable to supply-chain attacks if the tag is moved.

Failing references:
- build.yml: actions/checkout@v6.0.1, actions/setup-node@v6.2.0 (×2), codecov/codecov-action@v5, actions/upload-artifact@v6.0.0
- integration.yml: actions/checkout@v6.0.1, JamesIves/github-sponsors-readme-action@releases/v1 (×3)
- label.yml: actions/checkout@v6.0.1, mauroalderete/action-assign-labels@v1.5.1
- production.yml: actions/checkout@v6.0.1, actions/setup-node@v6.2.0
- sponsors.yml: actions/checkout@v6.0.1, JamesIves/github-sponsors-readme-action@v1 (×2), JamesIves/github-pages-deploy-action@v4
- version.yml: nowactions/update-majorver@v1.1.2, actions/checkout@v6.0.1, actions/setup-node@v6.2.0 (×2)

Locations:

- `.github/workflows/build.yml:17`
- `.github/workflows/build.yml:19`
- `.github/workflows/build.yml:35`
- `.github/workflows/build.yml:37`
- `.github/workflows/build.yml:48`
- `.github/workflows/build.yml:57`
- `.github/workflows/integration.yml:14`
- `.github/workflows/integration.yml:17`
- `.github/workflows/integration.yml:24`
- `.github/workflows/integration.yml:33`
- `.github/workflows/label.yml:17`
- `.github/workflows/label.yml:21`
- `.github/workflows/production.yml:16`
- `.github/workflows/production.yml:18`
- `.github/workflows/sponsors.yml:13`
- `.github/workflows/sponsors.yml:22`
- `.github/workflows/sponsors.yml:35`
- `.github/workflows/sponsors.yml:47`
- `.github/workflows/version.yml:11`
- `.github/workflows/version.yml:26`
- `.github/workflows/version.yml:32`
- `.github/workflows/version.yml:57`
- `.github/workflows/version.yml:61`

### missing-permissions (severity: medium)

Five workflow files have no top-level `permissions:` key and no job-level `permissions:` keys on any of their jobs. Without explicit permissions, workflows run with the repository's default token permissions (often write-all), violating the principle of least privilege. Only label.yml correctly declares explicit permissions.

Locations:

- `.github/workflows/build.yml:1`
- `.github/workflows/integration.yml:1`
- `.github/workflows/production.yml:1`
- `.github/workflows/sponsors.yml:1`
- `.github/workflows/version.yml:1`

### script-injection (severity: high)

GitHub Actions expressions (`${{ ... }}`) are interpolated directly inside `run:` shell command strings, violating rule (a). These values are substituted into the shell script before the shell parses it, allowing an attacker who controls the value to inject arbitrary shell commands.

- production.yml lines 42–44: `${{ secrets.GIT_CONFIG_EMAIL }}`, `${{ secrets.GIT_CONFIG_NAME }}`, and `${{ github.sha }}` are all interpolated directly in a `run:` block.
- version.yml line 63: `${{ secrets.GITHUB_TOKEN }}` is interpolated directly in `echo "//npm.pkg.github.com:_authToken=${{ secrets.GITHUB_TOKEN }}" > ~/.npmrc`.
- version.yml lines 51–52: `$VERSION` (derived from `$GITHUB_REF` which is set from `${{ github.ref }}`) is used unquoted in `npm version $VERSION`, violating rule (b) — unquoted shell variable expansion of workflow-controllable data.

Locations:

- `.github/workflows/production.yml:42`
- `.github/workflows/production.yml:43`
- `.github/workflows/production.yml:44`
- `.github/workflows/version.yml:52`
- `.github/workflows/version.yml:63`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all findings across 6 workflow files:

1. unpinned-uses: Pinned all action references to full 40-char SHAs with tag comments preserved:
   - actions/checkout@v6.0.1 → @8e8c483db84b4bee98b60c0593521ed34d9990e8
   - actions/setup-node@v6.2.0 → @6044e13b5dc448c55e2357c09f80417699197238
   - codecov/codecov-action@v5 → @0fb7174895f61a3b6b78fc075e0cd60383518dac
   - actions/upload-artifact@v6.0.0 → @b7c566a772e6b6bfb58ed0dc250532a479d7789f
   - JamesIves/github-sponsors-readme-action@releases/v1 and @v1 → @2fd9142e765f755780202122261dc85e78459405
   - mauroalderete/action-assign-labels@v1.5.1 → @671a4ca2da0f900464c58b8b5540a1e07133e915
   - JamesIves/github-pages-deploy-action@v4 → @d92aa235d04922e8f08b40ce78cc5442fcfbfa2f
   - nowactions/update-majorver@v1.1.2 → @f2014bbbba95b635e990ce512c5653bd0f4753fb

2. missing-permissions: Added top-level permissions blocks to build.yml (contents: read), integration.yml (contents: read), production.yml (contents: write), sponsors.yml (contents: write), and version.yml (contents: write).

3. script-injection: Moved all ${{ }} expressions out of run: blocks into env: blocks in production.yml (GIT_CONFIG_EMAIL, GIT_CONFIG_NAME, github.sha) and version.yml (secrets.GITHUB_TOKEN in npmrc step). Also quoted $VERSION in npm version command in version.yml.

