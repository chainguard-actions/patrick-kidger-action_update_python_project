<!-- markdownlint-disable -->

# Hardening Report: patrick-kidger--action_update_python_project/v7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **patrick-kidger--action_update_python_project/v7** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions into shell commands (sub-rule a). The most critical instance is `${{ inputs.test-script }}` interpolated directly inside `bash -c "..."` — this allows the calling workflow to inject arbitrary shell commands. Additional violations include `${{ github.workspace }}` used unquoted in pip install URLs and `cd` commands, `${{ inputs.github-user }}`, `${{ inputs.github-token }}`, and `${{ github.repository }}` embedded in a `git push` URL, and `${{ steps.get-versions.outputs.* }}` values echoed directly in the Logging step. All of these bypass shell quoting and allow metacharacter injection. Fix: move all expression values into `env:` variables and reference them as double-quoted shell variables (e.g. `"$ENV_VAR"`).

Locations:

- `action.yml:70`
- `action.yml:72`
- `action.yml:76`
- `action.yml:83`
- `action.yml:85`
- `action.yml:91`
- `action.yml:95`
- `action.yml:96`
- `action.yml:97`
- `action.yml:108`
- `action.yml:109`

### unpinned-uses (severity: high)

Three `uses:` references are pinned to mutable tags rather than immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved or the upstream repositories are compromised: `actions/checkout@v2` (line 27), `actions/setup-python@v2` (line 30), and `softprops/action-gh-release@v1` (line 113). Each should be replaced with a full SHA pin, e.g. `actions/checkout@<40-char-sha> # v2`.

Locations:

- `action.yml:27`
- `action.yml:30`
- `action.yml:113`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.test-script }}" appears directly in run: block of step "Test sdist"; move to env: map

Locations:

- `action.yml:83`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.test-script }}" appears directly in run: block of step "Test bdist_wheel"; move to env: map

Locations:

- `action.yml:104`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.github-user }}" appears directly in run: block of step "Tag"; move to env: map

Locations:

- `action.yml:139`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.github-token }}" appears directly in run: block of step "Tag"; move to env: map

Locations:

- `action.yml:139`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, static-inline-injection

**Notes:**

Fixed all security findings in hardened/action/action.yml:

1. **unpinned-uses**: Pinned all three `uses:` references to immutable commit SHAs:
   - `actions/checkout@v2` → `@0717577d45739eb3c851188b29f50ed6c0b2194e # v2`
   - `actions/setup-python@v2` → `@e9aba2c848f5ebd159c070c61ea2c4e2b122355e # v2`
   - `softprops/action-gh-release@v1` → `@de2c0eb89ae2a093876385947365aca7b0e5f844 # v1`

2. **script-injection / static-inline-injection**: Moved all `${{ ... }}` expressions from `run:` blocks into `env:` maps and referenced them as double-quoted shell variables:
   - **Test sdist**: `${{ github.workspace }}` → `GITHUB_WORKSPACE_PATH`, `${{ inputs.test-script }}` → `INPUT_TEST_SCRIPT`
   - **Test bdist_wheel**: same as above
   - **Logging**: `${{ steps.get-versions.outputs.new-version }}` → `NEW_VERSION`, etc.
   - **Tag**: `${{ inputs.github-user }}` → `GITHUB_USER`, `${{ inputs.github-token }}` → `GITHUB_TOKEN_VALUE`, `${{ github.repository }}` → `GITHUB_REPOSITORY_VALUE`, `${{ steps.get-versions.outputs.tag }}` → `GIT_TAG`

All shell variables are properly double-quoted in the run: scripts.

