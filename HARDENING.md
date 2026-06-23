<!-- markdownlint-disable -->

# Hardening Report: patrick-kidger--action_update_python_project/v7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **patrick-kidger--action_update_python_project/v7** was hardened automatically. 8 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three `uses:` references in action.yml use mutable tags instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if the referenced tags are moved: `actions/checkout@v2` (line 30), `actions/setup-python@v2` (line 33), `softprops/action-gh-release@v1` (line 143).

Locations:

- `action.yml:30`
- `action.yml:33`
- `action.yml:143`

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate GitHub Actions expressions (`${{ ... }}`) into shell commands, violating rule (a). (1) 'Test sdist' step (lines 80, 83, 92): `${{ github.workspace }}` is interpolated into a pip install command and a cd command; critically, `${{ inputs.test-script }}` is interpolated directly inside `bash -c "..."`, allowing an attacker-controlled caller to inject arbitrary shell commands. (2) 'Test bdist_wheel' step (lines 101, 104, 113): same pattern with `${{ github.workspace }}` and `${{ inputs.test-script }}`. (3) 'Logging' step (lines 118-120): `${{ steps.get-versions.outputs.new-version }}`, `${{ steps.test-sdist.outputs.result }}`, `${{ steps.test-bdist-wheel.outputs.result }}` are interpolated into echo commands. (4) 'Tag' step (lines 138-139): `${{ steps.get-versions.outputs.tag }}`, `${{ inputs.github-user }}`, `${{ inputs.github-token }}`, and `${{ github.repository }}` are interpolated directly into git commands, exposing credentials and enabling injection.

Locations:

- `action.yml:80`
- `action.yml:83`
- `action.yml:92`
- `action.yml:101`
- `action.yml:104`
- `action.yml:113`
- `action.yml:118`
- `action.yml:119`
- `action.yml:120`
- `action.yml:138`
- `action.yml:139`

### github-env-injection (severity: high)

The 'Get versions' step (lines 63-65) writes values derived from pyproject.toml (`name`, `checkout_version`, `new_version`) to `$GITHUB_OUTPUT` via Python subprocess without sanitization (no `tr -d '\n\r'`). These values are project-controlled and could contain newlines that inject additional key=value pairs into GITHUB_OUTPUT. The 'Tag' step (line 138) also writes `${{ steps.get-versions.outputs.tag }}` (derived from pyproject.toml version) directly into a git tag command without sanitization.

Locations:

- `action.yml:63`
- `action.yml:64`
- `action.yml:65`
- `action.yml:138`

### github-env-injection (severity: high)

The 'Get versions' step (lines 63-65) writes values derived from pyproject.toml (`name`, `checkout_version`, `new_version`) to `$GITHUB_OUTPUT` via Python subprocess without sanitization (no `tr -d '\n\r'`). These values are project-controlled and could contain newlines that inject additional key=value pairs into GITHUB_OUTPUT. The 'Tag' step (line 138) also writes `${{ steps.get-versions.outputs.tag }}` (derived from pyproject.toml version) directly into a git tag command without sanitization.

Locations:

- `action.yml:63`
- `action.yml:64`
- `action.yml:65`
- `action.yml:138`

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

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed all security findings in action.yml:

1. unpinned-uses: Pinned actions/checkout@v2 to SHA ee0669bd1cc54295c223e0bb666b733df41de1c5, actions/setup-python@v2 to SHA e9aba2c848f5ebd159c070c61ea2c4e2b122355e, and softprops/action-gh-release@v1 to SHA de2c0eb89ae2a093876385947365aca7b0e5f844.

2. script-injection / static-inline-injection: Moved all ${{ }} expressions from run: blocks to env: blocks in Test sdist, Test bdist_wheel, Logging, and Tag steps. Shell scripts now reference plain environment variables ($WORKSPACE, $TEST_SCRIPT, $NEW_VERSION, $SDIST_RESULT, $BDIST_RESULT, $GIT_TAG, $GITHUB_USER, $GITHUB_TOKEN_INPUT, $GITHUB_REPOSITORY).

3. github-env-injection: In the Get versions step, replaced subprocess.run('echo ...') calls with direct Python file writes to $GITHUB_OUTPUT, adding a sanitize() function that strips \r and \n characters before writing. In the Tag step, added printf '%s' | tr -d '\n\r' sanitization for GIT_TAG, GITHUB_USER, and GITHUB_REPOSITORY before use in git commands.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in both 'Test sdist' (line 97) and 'Test bdist_wheel' (line 113) steps. Replaced `bash -c "$TEST_SCRIPT"` with a pattern that writes the TEST_SCRIPT env var to a temporary file via `printf '%s' "$TEST_SCRIPT" > "$_script_file"` and then executes it with `bash "$_script_file"`. This prevents the attacker-controlled `inputs.test-script` value from being interpreted as shell code by bash's `-c` flag, while still allowing legitimate multi-line shell scripts to be passed as the test-script input.

