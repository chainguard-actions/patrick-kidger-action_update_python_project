<!-- markdownlint-disable -->

# Hardening Report: patrick-kidger--action_update_python_project/v8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **patrick-kidger--action_update_python_project/v8** was hardened automatically. 10 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three `uses:` references are pinned to mutable tags rather than full 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved or hijacked:
- `actions/checkout@v2` (line 42)
- `actions/setup-python@v2` (line 46)
- `softprops/action-gh-release@v1` (line 157)
All should be pinned to a full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v2`.

Locations:

- `action.yml:42`
- `action.yml:46`
- `action.yml:157`

### script-injection (severity: high)

Multiple `${{ ... }}` expressions are interpolated directly inside `run:` shell blocks, violating rule (a). The most critical instance is `${{ inputs.test-script }}` being expanded verbatim inside `bash -c "..."` in both the 'Test sdist' and 'Test bdist_wheel' steps — a caller can supply arbitrary shell commands as the test-script input, achieving full remote code execution. Additional violations:

**'Get versions' step (lines 73, 80):** `${{ inputs.pypi-repository-url }}` and `${{ inputs.allow-first-release }}` are interpolated directly into a Python string literal passed to `python -c`, allowing injection of arbitrary Python code.

**'Test sdist' step (lines 96, 99, 107):** `${{ github.workspace }}` unquoted in a pip install path; `${{ inputs.test-script }}` expanded directly inside `bash -c`; `${{ github.workspace }}` unquoted in `cd`.

**'Test bdist_wheel' step (lines 113, 116, 124):** Same pattern as Test sdist.

**'Logging' step (lines 128–130):** `${{ steps.get-versions.outputs.new-version }}`, `${{ steps.test-sdist.outputs.result }}`, `${{ steps.test-bdist-wheel.outputs.result }}` interpolated directly into echo commands.

**'Push to PyPI' step (lines 140, 142):** `${{ inputs.pypi-repository-url }}` used unquoted in `if [ ${{ inputs.pypi-repository-url }} == https://pypi.org/ ]` and `export REPO_URL=${{ inputs.pypi-repository-url }}`, allowing shell metacharacter injection.

**'Tag' step (lines 149–150):** `${{ steps.get-versions.outputs.tag }}`, `${{ inputs.github-user }}`, `${{ inputs.github-token }}`, and `${{ github.repository }}` all interpolated directly into the `git push` URL and `git tag` command.

Locations:

- `action.yml:73`
- `action.yml:80`
- `action.yml:96`
- `action.yml:99`
- `action.yml:107`
- `action.yml:113`
- `action.yml:116`
- `action.yml:124`
- `action.yml:128`
- `action.yml:140`
- `action.yml:142`
- `action.yml:149`
- `action.yml:150`

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

**Fixes applied:** unpinned-uses, script-injection, static-inline-injection

**Notes:**

Fixed all security findings in hardened/action/action.yml:

1. Pinned all three uses: references to full 40-char SHAs:
   - actions/checkout@v2 → @0717577d45739eb3c851188b29f50ed6c0b2194e # v2
   - actions/setup-python@v2 → @e9aba2c848f5ebd159c070c61ea2c4e2b122355e # v2
   - softprops/action-gh-release@v1 → @de2c0eb89ae2a093876385947365aca7b0e5f844 # v1

2. Moved all ${{ }} expressions from run: blocks to env: blocks:
   - Get versions: pypi-repository-url and allow-first-release moved to env vars, read via os.environ in Python
   - Test sdist: github.workspace → GITHUB_WORKSPACE_PATH, inputs.test-script → INPUT_TEST_SCRIPT (written to temp file, run with bash -eo pipefail to preserve errexit)
   - Test bdist_wheel: same pattern as Test sdist
   - Logging: all three step outputs moved to env vars
   - Push to PyPI: pypi-repository-url → INPUT_PYPI_REPOSITORY_URL, properly quoted in shell
   - Tag: tag, github-user, github-token, github.repository all moved to env vars, used with proper quoting in git push URL

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell variable expansion in action.yml line 174: changed `--repository-url $REPO_URL` to `--repository-url "$REPO_URL"` in the 'Push to PyPI' step's twine upload command. The variable `REPO_URL` is derived from `inputs.pypi-repository-url` (a workflow-controllable value), so leaving it unquoted allowed shell metacharacters to be interpreted, enabling command injection.

