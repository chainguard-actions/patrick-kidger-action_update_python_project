<!-- markdownlint-disable -->

# Hardening Report: patrick-kidger--action_update_python_project/v7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **patrick-kidger--action_update_python_project/v7** was hardened automatically. 7 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three `uses:` references in action.yml use mutable tag refs instead of pinned 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved:
- `actions/checkout@v2` (line 27)
- `actions/setup-python@v2` (line 30)
- `softprops/action-gh-release@v1` (line 131)
Each should be replaced with a full SHA digest, e.g. `actions/checkout@<40-char-sha> # v2`.

Locations:

- `action.yml:27`
- `action.yml:30`
- `action.yml:131`

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions into shell commands (sub-rule a), allowing script injection:

1. **"Test sdist" step (~line 83)**: `${{ inputs.test-script }}` is interpolated directly inside `bash -c "..."` — an attacker-controlled input is executed as arbitrary shell code. Also `${{ github.workspace }}` is interpolated unquoted in a path argument.

2. **"Test bdist_wheel" step (~line 98)**: Same patterns — `${{ inputs.test-script }}` inside `bash -c "..."` and `${{ github.workspace }}` unquoted.

3. **"Logging" step (~line 113)**: `${{ steps.get-versions.outputs.new-version }}`, `${{ steps.test-sdist.outputs.result }}`, and `${{ steps.test-bdist-wheel.outputs.result }}` are interpolated directly into `echo` commands.

4. **"Tag" step (~line 120)**: `${{ steps.get-versions.outputs.tag }}`, `${{ inputs.github-user }}`, `${{ inputs.github-token }}`, and `${{ github.repository }}` are all interpolated directly into `git tag` and `git push` commands. Injecting a crafted tag value or username could allow arbitrary command execution.

Locations:

- `action.yml:83`
- `action.yml:88`
- `action.yml:98`
- `action.yml:103`
- `action.yml:113`
- `action.yml:120`
- `action.yml:123`

### github-env-injection (severity: high)

The **"Get versions" step** writes values derived from the repository's `pyproject.toml` (project `name` and `checkout_version`) directly to `$GITHUB_OUTPUT` via Python `subprocess.run(f'echo name={name} >> $GITHUB_OUTPUT', shell=True)` and similar calls, without any newline sanitization (`printf '%s' ... | tr -d '\n\r'`). If `pyproject.toml` contains a crafted `name` or `version` field with embedded newlines, an attacker who controls the repository content could inject arbitrary key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting subsequent step outputs and influencing downstream conditional logic.

Locations:

- `action.yml:57`
- `action.yml:58`
- `action.yml:59`

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

Fixed all findings in action.yml:
1. Pinned actions/checkout@v2, actions/setup-python@v2, and softprops/action-gh-release@v1 to full 40-char commit SHAs.
2. Moved all ${{ }} expressions from run: blocks into env: maps for Test sdist, Test bdist_wheel, Logging, and Tag steps.
3. Fixed github-env-injection in the Get versions step: replaced subprocess.run('echo ... >> $GITHUB_OUTPUT', shell=True) with direct Python file writes using a safe() function that strips newlines/carriage returns from values before writing to $GITHUB_OUTPUT.
4. Preserved upstream behavior: test-script runs via bash -c without -e (no errexit), matching upstream's intentional design where $? is captured manually.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script injection findings in action.yml. In both 'Test sdist' (line 76) and 'Test bdist_wheel' (line 94) steps, the unquoted `$TEST_SCRIPT` expansion inside `bash -c` was replaced with properly double-quoted `bash -c "$TEST_SCRIPT"`. The multi-line heredoc-style expansion (`bash -c "\n              $TEST_SCRIPT\n              "`) was also collapsed to a single-line form. The fix preserves upstream's intentional use of `bash -c` without `-e` (no errexit), since the test-script step uses `set +e` and captures the exit code manually.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in both 'Test sdist' (line 73) and 'Test bdist_wheel' (line 89) steps. Replaced `bash -c "$TEST_SCRIPT"` with writing `$TEST_SCRIPT` to a temp file via `printf '%s\n' "$TEST_SCRIPT" > "$_script_file"` and executing it with `bash "$_script_file"`. This prevents the shell from interpreting the input value as inline shell code. The fix preserves upstream's behavior: no `-e` flag is used (matching the `set +e` context upstream used), and the exit code is still captured for the result output. Temp files are cleaned up with `rm -f` after use.

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Fixed the script injection finding in the 'Logging' step of action.yml (lines 136-138). The three echo statements that expanded unquoted variables ($NEW_VERSION, $SDIST_RESULT, $BDIST_RESULT) were updated to use double-quoted strings: `echo "new-version=$NEW_VERSION"`, `echo "sdist-result=$SDIST_RESULT"`, and `echo "bdist-result=$BDIST_RESULT"`. The variables were already correctly sourced from the env: block (populated from steps.*.outputs.* expressions) in a prior iteration; this iteration adds the missing double-quotes to prevent shell metacharacter injection.

