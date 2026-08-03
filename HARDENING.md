<!-- markdownlint-disable -->

# Hardening Report: richardsimko--update-tag/v3.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **richardsimko--update-tag/v3.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All workflow files use mutable version tags instead of pinned 40-character SHA commit hashes for `uses:` references, making them vulnerable to supply-chain attacks if the referenced action tags are moved.

auto-merge.yml: `actions/checkout@v7`, `dependabot/fetch-metadata@v3`
master-ci.yml: `actions/checkout@v7`, `actions/setup-node@v7`
publish.yml: `actions/checkout@v7`, `actions/setup-node@v7`, `planetscale/ghcommit-action@v0.2.22` (×2), `mikepenz/release-changelog-builder-action@v6`, `softprops/action-gh-release@v3`
test-released-version.yml: `actions/checkout@v7`, `actions/setup-node@v7`, `richardsimko/update-tag@v2`
update-major-version.yml: `actions/checkout@v7`, `richardsimko/update-tag@v2` (×2)

Locations:

- `.github/workflows/auto-merge.yml:22`
- `.github/workflows/auto-merge.yml:25`
- `.github/workflows/master-ci.yml:14`
- `.github/workflows/master-ci.yml:16`
- `.github/workflows/publish.yml:25`
- `.github/workflows/publish.yml:27`
- `.github/workflows/publish.yml:42`
- `.github/workflows/publish.yml:75`
- `.github/workflows/publish.yml:91`
- `.github/workflows/publish.yml:107`
- `.github/workflows/test-released-version.yml:11`
- `.github/workflows/test-released-version.yml:13`
- `.github/workflows/test-released-version.yml:18`
- `.github/workflows/update-major-version.yml:11`
- `.github/workflows/update-major-version.yml:18`
- `.github/workflows/update-major-version.yml:24`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate GitHub Actions expressions (`${{ ... }}`) into shell commands, enabling script injection. An attacker who controls the interpolated values (e.g. via a PR title, PR number, or workflow_dispatch input) can inject arbitrary shell commands.

auto-merge.yml (sub-rule a):
- `gh pr merge --auto --squash --subject "${{ github.event.pull_request.title }} (#${{ github.event.pull_request.number }})" ... ${{ github.event.pull_request.number }}` — PR title and number are attacker-controlled via pull_request_target.
- `gh api ... /repos/${{ github.repository }}/pulls/${{ github.event.pull_request.number }}/... -f "reviewers[]=${{ env.REVIEWER }}"` — PR number interpolated directly.

publish.yml (sub-rule a):
- `echo "version=$(npm version --no-git-tag-version ${{ inputs.versionType }})"` — workflow_dispatch input interpolated directly.
- `git checkout -b version-bump/${{steps.update-version.outputs.version}}` and `git push origin version-bump/${{steps.update-version.outputs.version}}` — step output interpolated directly.
- `gh pr create --head version-bump/${{steps.update-version.outputs.version}} --title "... ${{steps.update-version.outputs.version}}" --body "...${{steps.update-version.outputs.version}}"` — step output interpolated directly.
- `git checkout -b release/${{steps.update-version.outputs.version}}` and `git push origin release/${{steps.update-version.outputs.version}}` — step output interpolated directly.
- `git tag ${{steps.update-version.outputs.version}} ${{ steps.commit.outputs.commit-hash }}` — step outputs interpolated directly.

Locations:

- `.github/workflows/auto-merge.yml:30`
- `.github/workflows/auto-merge.yml:40`
- `.github/workflows/publish.yml:36`
- `.github/workflows/publish.yml:40`
- `.github/workflows/publish.yml:48`
- `.github/workflows/publish.yml:57`
- `.github/workflows/publish.yml:96`

### missing-permissions (severity: medium)

These workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, the GITHUB_TOKEN is granted default (potentially broad) permissions, violating the principle of least privilege.

- test-released-version.yml: No permissions defined at any level.
- update-major-version.yml: No permissions defined at any level.

Locations:

- `.github/workflows/test-released-version.yml:1`
- `.github/workflows/update-major-version.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all 5 workflow files:

1. **unpinned-uses**: Pinned all action references to full SHA hashes: actions/checkout@v7→3d3c42e5..., dependabot/fetch-metadata@v3→25dd0e34..., actions/setup-node@v7→820762786..., planetscale/ghcommit-action@v0.2.22→a6b150b8..., mikepenz/release-changelog-builder-action@v6→c9bcd823..., softprops/action-gh-release@v3→3d0d9888..., richardsimko/update-tag@v2→d43d3236...

2. **script-injection**: Moved all ${{ }} expressions out of run: blocks into env: blocks in auto-merge.yml (PR title, PR number, repository) and publish.yml (versionType input, version output, commit-hash output). Shell scripts now reference plain $VAR environment variables.

3. **missing-permissions**: Added `permissions: contents: write` to test-released-version.yml and update-major-version.yml at the top level (minimum needed for tag operations).

