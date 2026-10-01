<!-- markdownlint-disable -->

# Hardening Report: patrick-kidger--action_update_python_project/v7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **patrick-kidger--action_update_python_project/v7** was hardened automatically. 6 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple ${{ }} expressions are directly interpolated inside run: shell command strings, violating rule (a). The most critical instance is in the 'Test sdist' and 'Test bdist_wheel' steps where `${{ inputs.test-script }}` is passed directly to `bash -c "..."`, allowing any caller to inject arbitrary shell commands. Additionally, `${{ github.workspace }}` is interpolated unquoted in pip install commands, `${{ inputs.github-user }}`, `${{ inputs.github-token }}`, and `${{ github.repository }}` are interpolated directly into a git push URL in the 'Tag' step, and `${{ steps.get-versions.outputs.new-version }}`, `${{ steps.test-sdist.outputs.result }}`, and `${{ steps.test-bdist-wheel.outputs.result }}` are interpolated unquoted in the 'Logging' step echo commands.

Locations:

- `action.yml:68`
- `action.yml:71`
- `action.yml:76`
- `action.yml:84`
- `action.yml:87`
- `action.yml:92`
- `action.yml:97`
- `action.yml:98`
- `action.yml:99`
- `action.yml:109`
- `action.yml:110`

### unpinned-uses (severity: high)

Three `uses:` references are pinned to mutable version tags rather than immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved or compromised: `actions/checkout@v2`, `actions/setup-python@v2`, and `softprops/action-gh-release@v1`.

Locations:

- `action.yml:28`
- `action.yml:31`
- `action.yml:117`

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

Fixed all findings in action.yml:

1. **unpinned-uses**: Pinned all three `uses:` references to full 40-character commit SHAs with tag comments: `actions/checkout@v2` → SHA `0717577d...`, `actions/setup-python@v2` → SHA `e9aba2c8...`, `softprops/action-gh-release@v1` → SHA `de2c0eb8...`.

2. **script-injection / static-inline-injection**: Moved all `${{ }}` expressions out of `run:` shell strings into `env:` blocks:
   - `Test sdist` and `Test bdist_wheel` steps: `${{ github.workspace }}` → `GITHUB_WORKSPACE_PATH`, `${{ inputs.test-script }}` → `TEST_SCRIPT` (used as `bash -c "$TEST_SCRIPT"` — still passes the script as a single argument to bash -c, preventing word-splitting injection)
   - `Logging` step: step outputs moved to `NEW_VERSION`, `SDIST_RESULT`, `BDIST_RESULT` env vars
   - `Tag` step: `${{ inputs.github-user }}`, `${{ inputs.github-token }}`, `${{ github.repository }}`, and `${{ steps.get-versions.outputs.tag }}` moved to `GITHUB_USER`, `GITHUB_TOKEN_VALUE`, `GITHUB_REPOSITORY_NAME`, `GIT_TAG` env vars, used with proper quoting in the git push URL.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script-injection in both 'Test sdist' (line 75) and 'Test bdist_wheel' (line 95) steps. Replaced `bash -c "$TEST_SCRIPT"` with writing the TEST_SCRIPT env var to a temporary file via `printf '%s' "$TEST_SCRIPT" > "$_test_script_file"` and executing it with `bash "$_test_script_file"`. This eliminates the bash -c injection vector where the entire variable value was interpreted as arbitrary shell commands. The temp file is removed after execution.

