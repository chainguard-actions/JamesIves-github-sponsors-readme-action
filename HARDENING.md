<!-- markdownlint-disable -->

# Hardening Report: JamesIves--github-sponsors-readme-action/v1.6.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **JamesIves--github-sponsors-readme-action/v1.6.1** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references across every workflow file use mutable version tags instead of pinned 40-character SHA commits. This exposes the workflows to supply-chain attacks if any referenced action's tag is moved or compromised. Affected references include: actions/checkout@v7.0.1, actions/setup-node@v7.0.0, codecov/codecov-action@v7.0.0, actions/upload-artifact@v7.0.1, mauroalderete/action-assign-labels@v1.5.1, nowactions/update-majorver@v1.1.2, JamesIves/github-sponsors-readme-action@v1, JamesIves/github-pages-deploy-action@v4.

Locations:

- `.github/workflows/build.yml:20`
- `.github/workflows/build.yml:22`
- `.github/workflows/build.yml:37`
- `.github/workflows/build.yml:39`
- `.github/workflows/build.yml:46`
- `.github/workflows/integration.yml:27`
- `.github/workflows/integration.yml:32`
- `.github/workflows/integration.yml:73`
- `.github/workflows/integration.yml:78`
- `.github/workflows/label.yml:16`
- `.github/workflows/label.yml:21`
- `.github/workflows/production.yml:32`
- `.github/workflows/production.yml:34`
- `.github/workflows/release.yml:43`
- `.github/workflows/release.yml:97`
- `.github/workflows/sponsors.yml:16`
- `.github/workflows/sponsors.yml:22`
- `.github/workflows/sponsors.yml:33`
- `.github/workflows/sponsors.yml:44`
- `.github/workflows/version.yml:14`
- `.github/workflows/version.yml:34`
- `.github/workflows/version.yml:38`
- `.github/workflows/version.yml:57`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ ... }}` expressions into shell commands, enabling script injection. (a) release.yml: `${{ inputs.bump }}` is interpolated directly into `npm --no-git-tag-version version "${{ inputs.bump }}"` — a workflow_dispatch input fully controlled by the caller. (a) release.yml: `${{ steps.version.outputs.target-branch }}` is interpolated unquoted into `git checkout -b ${{ steps.version.outputs.target-branch }}` and `git push origin ${{ steps.version.outputs.target-branch }}` and `git fetch/checkout` commands. (a) release.yml: `${{ needs.prepare.outputs.next-version }}` and `${{ needs.prepare.outputs.target-branch }}` are interpolated into the `gh release create` shell command. (a) integration.yml: `${{ steps.skipped.outputs.sponsorshipStatus }}` is interpolated into an `echo` command. (a) production.yml: `${{ github.sha }}` is interpolated into a `git commit -m` message string.

Locations:

- `.github/workflows/release.yml:30`
- `.github/workflows/release.yml:57`
- `.github/workflows/release.yml:66`
- `.github/workflows/release.yml:72`
- `.github/workflows/release.yml:113`
- `.github/workflows/integration.yml:89`
- `.github/workflows/production.yml:50`

### github-env-injection (severity: high)

In release.yml, the `Compute the next version and target release branch` step writes `NEXT_VERSION` and `NEXT_MAJOR` to `$GITHUB_OUTPUT` without sanitization. These variables are derived from `${{ inputs.bump }}`, a user-controlled `workflow_dispatch` input. An attacker-supplied value containing newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT (e.g., `next-version=injected\nsome-other-key=malicious`). The required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`) is absent before the `echo ... >> "$GITHUB_OUTPUT"` writes.

Locations:

- `.github/workflows/release.yml:57`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three finding types across 6 workflow files:

1. unpinned-uses: Pinned all 8 distinct action references to their full 40-char SHA commits across build.yml, integration.yml, label.yml, production.yml, release.yml, sponsors.yml, and version.yml. Original tags preserved as inline comments.

2. script-injection: Moved all ${{ }} expressions out of run: shell commands into env: blocks. Fixed in release.yml (inputs.bump, steps.version.outputs.target-branch in two steps, needs.prepare.outputs.next-version/target-branch, and secrets), integration.yml (steps.skipped.outputs.sponsorshipStatus), and production.yml (github.sha).

3. github-env-injection: In release.yml's 'Compute the next version' step, added sanitization using `printf '%s' "$VAR" | tr -d '\n\r'` before writing NEXT_VERSION and NEXT_MAJOR (derived from user-controlled inputs.bump) to $GITHUB_OUTPUT.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings in .github/workflows/version.yml:
1. (line 60) Quoted `$VERSION` in `npm version "$VERSION"` to prevent shell metacharacter injection from the workflow-controllable GITHUB_REF value.
2. (line 79) Moved `${{ secrets.GITHUB_TOKEN }}` from the `run:` shell command string into an `env:` block (`GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}`), then referenced it as `$GITHUB_TOKEN` in the shell script to prevent direct interpolation into the shell command.

