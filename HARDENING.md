<!-- markdownlint-disable -->

# Hardening Report: patrick-kidger--action_update_python_project/v8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **patrick-kidger--action_update_python_project/v8** was hardened automatically. 11 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three `uses:` references in action.yml use mutable version tags instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if the upstream tag is moved or compromised: `actions/checkout@v2` (line 43), `actions/setup-python@v2` (line 46), and `softprops/action-gh-release@v1` (line 155). Each should be pinned to a full commit SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `action.yml:43`
- `action.yml:46`
- `action.yml:155`

### script-injection (severity: high)

Multiple `${{ }}` expressions are interpolated directly inside `run:` shell blocks (rule a), allowing script injection:

(1) 'Get versions' step (lines ~75, ~82): `${{ inputs.pypi-repository-url }}` and `${{ inputs.allow-first-release }}` are embedded verbatim inside a Python string passed to `python -c "..."`. An attacker-controlled value containing single-quotes or Python metacharacters can break out of the string and execute arbitrary Python code.

(2) 'Test sdist' and 'Test bdist_wheel' steps (lines ~101, ~103, ~111, ~119, ~121, ~129): `${{ github.workspace }}` is interpolated unquoted in pip install and `cd` commands; critically, `${{ inputs.test-script }}` is interpolated directly inside `bash -c "..."`, meaning the entire content of the `test-script` input is executed as shell code — this is an intentional design but still constitutes a script-injection vector since the input value is not sanitized before shell evaluation.

(3) 'Logging' step (lines ~133–135): `${{ steps.get-versions.outputs.new-version }}`, `${{ steps.test-sdist.outputs.result }}`, and `${{ steps.test-bdist-wheel.outputs.result }}` are interpolated directly into `echo` commands.

(4) 'Push to PyPI' step (lines ~140, ~142): `${{ inputs.pypi-repository-url }}` is interpolated unquoted in `if [ ${{ inputs.pypi-repository-url }} == https://pypi.org/ ]` and `export REPO_URL=${{ inputs.pypi-repository-url }}`, allowing shell metacharacter injection.

(5) 'Tag' step (lines ~149–150): `${{ steps.get-versions.outputs.tag }}`, `${{ inputs.github-user }}`, `${{ inputs.github-token }}`, and `${{ github.repository }}` are all interpolated directly into git commands, allowing injection into the git tag name and the push URL.

Locations:

- `action.yml:75`
- `action.yml:82`
- `action.yml:101`
- `action.yml:103`
- `action.yml:111`
- `action.yml:119`
- `action.yml:121`
- `action.yml:129`
- `action.yml:133`
- `action.yml:134`
- `action.yml:135`
- `action.yml:140`
- `action.yml:142`
- `action.yml:149`
- `action.yml:150`

### github-env-injection (severity: high)

The 'Get versions' step writes values to `$GITHUB_OUTPUT` via Python `subprocess.run(f'echo name={name} >> $GITHUB_OUTPUT', shell=True)` and similar calls (lines ~87–89). The values `name` and `checkout_version` are read from `pyproject.toml` without sanitization — a newline character in either value would allow injecting additional key=value pairs into GITHUB_OUTPUT (header injection). No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before any of these writes. Additionally, the 'Test sdist' and 'Test bdist_wheel' steps write literal `result=true` / `result=false` to `$GITHUB_OUTPUT` (safe), but the 'Logging' step echoes `${{ steps.get-versions.outputs.new-version }}` etc. directly to stdout without writing to GITHUB_OUTPUT (not a direct injection, but confirms unsanitized data flows).

Locations:

- `action.yml:87`
- `action.yml:88`
- `action.yml:89`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.pypi-repository-url }}" appears directly in run: block of step "Get versions"; move to env: map

Locations:

- `action.yml:72`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.allow-first-release }}" appears directly in run: block of step "Get versions"; move to env: map

Locations:

- `action.yml:81`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.test-script }}" appears directly in run: block of step "Test sdist"; move to env: map

Locations:

- `action.yml:106`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.test-script }}" appears directly in run: block of step "Test bdist_wheel"; move to env: map

Locations:

- `action.yml:127`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.pypi-repository-url }}" appears directly in run: block of step "Push to PyPI"; move to env: map

Locations:

- `action.yml:158`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.pypi-repository-url }}" appears directly in run: block of step "Push to PyPI"; move to env: map

Locations:

- `action.yml:161`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.github-user }}" appears directly in run: block of step "Tag"; move to env: map

Locations:

- `action.yml:172`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.github-token }}" appears directly in run: block of step "Tag"; move to env: map

Locations:

- `action.yml:172`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-inline-injection

**Notes:**

Rewrote action.yml with all security fixes:

1. **unpinned-uses**: Pinned all three `uses:` references to full commit SHAs:
   - `actions/checkout@v2` → `@0717577d45739eb3c851188b29f50ed6c0b2194e # v2`
   - `actions/setup-python@v2` → `@e9aba2c848f5ebd159c070c61ea2c4e2b122355e # v2`
   - `softprops/action-gh-release@v1` → `@de2c0eb89ae2a093876385947365aca7b0e5f844 # v1`

2. **script-injection / static-inline-injection**: All `${{ }}` expressions moved to `env:` blocks:
   - `Get versions`: `inputs.pypi-repository-url` and `inputs.allow-first-release` moved to env vars, read via `os.environ` in Python
   - `Test sdist` & `Test bdist_wheel`: `inputs.test-script` moved to `INPUT_TEST_SCRIPT` env var, written to temp file and run with `bash "$_test_script_file"` (no errexit, matching upstream's intentional design); `github.workspace` replaced with built-in `$GITHUB_WORKSPACE`
   - `Logging`: step outputs moved to env vars
   - `Push to PyPI`: `inputs.pypi-repository-url` moved to `INPUT_PYPI_REPOSITORY_URL` env var
   - `Tag`: all expressions (tag, github-user, github-token, github.repository) moved to env vars

3. **github-env-injection**: Values written to `$GITHUB_OUTPUT` in the Python script are sanitized by stripping `\n` and `\r` characters before writing, preventing header injection via malicious pyproject.toml values.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script injection vulnerabilities in action.yml:
1. (Line 155, Push to PyPI step) Quoted the `$REPO_URL` variable in the twine upload command: changed `--repository-url $REPO_URL` to `--repository-url "$REPO_URL"` to prevent word splitting and glob expansion from an attacker-controlled URL.
2. (Line 76, Get versions step) Replaced the Python `subprocess.run()` call that used `shell=True` with an f-string interpolating `repo_url` directly into a shell command string. Changed it to use a list of arguments with `shell=False`, so the package name and repository URL are passed as literal arguments rather than being interpreted by the shell. This eliminates the shell injection vector from attacker-controlled `inputs.pypi-repository-url` values.

