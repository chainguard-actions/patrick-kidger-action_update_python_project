<!-- markdownlint-disable -->

# Hardening Report: patrick-kidger--action_update_python_project/v8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **patrick-kidger--action_update_python_project/v8** was hardened automatically. 10 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three `uses:` references in action.yml are pinned to mutable version tags rather than immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved or hijacked:
- `actions/checkout@v2` (line 44)
- `actions/setup-python@v2` (line 48)
- `softprops/action-gh-release@v1` (line 163)
Each should be replaced with the full SHA of the desired commit, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `action.yml:44`
- `action.yml:48`
- `action.yml:163`

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions into shell commands (rule a), allowing an attacker who controls those inputs to inject arbitrary shell commands.

(1) **Test sdist / Test bdist_wheel steps** — `${{ inputs.test-script }}` is interpolated directly inside `bash -c "${{ inputs.test-script }}"`. This is the most critical instance: the entire test-script input is executed as a shell command, giving any caller of this composite action full code execution on the runner.
  Offending lines (Test sdist): `python -m uv pip install ${{ github.workspace }}/dist/*.tar.gz ...` and `${{ inputs.test-script }}`
  Offending lines (Test bdist_wheel): same pattern.

(2) **Get versions step** — `${{ inputs.pypi-repository-url }}` and `${{ inputs.allow-first-release }}` are interpolated directly into a Python script string embedded in the shell `run:` block.

(3) **Push to PyPI step** — `${{ inputs.pypi-repository-url }}` is interpolated unquoted into a shell `if [ ${{ inputs.pypi-repository-url }} == https://pypi.org/ ]` test and into `export REPO_URL=${{ inputs.pypi-repository-url }}`.

(4) **Tag step** — `${{ inputs.github-user }}`, `${{ inputs.github-token }}`, and `${{ github.repository }}` are interpolated directly into a `git push https://...@github.com/...` URL, and `${{ steps.get-versions.outputs.tag }}` is interpolated into `git tag`.

(5) **Logging step** — `${{ steps.get-versions.outputs.new-version }}`, `${{ steps.test-sdist.outputs.result }}`, and `${{ steps.test-bdist-wheel.outputs.result }}` are interpolated directly into `echo` commands.

All of these must be moved to `env:` variables and then referenced as quoted shell variables (e.g. `"$VAR"`) in the `run:` script.

Locations:

- `action.yml:75`
- `action.yml:78`
- `action.yml:84`
- `action.yml:97`
- `action.yml:100`
- `action.yml:106`
- `action.yml:119`
- `action.yml:122`
- `action.yml:128`
- `action.yml:140`
- `action.yml:143`
- `action.yml:144`
- `action.yml:155`
- `action.yml:157`
- `action.yml:160`

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

Fixed all findings in hardened/action/action.yml:

1. Pinned three unpinned `uses:` references to full commit SHAs:
   - actions/checkout@v2 → @0717577d45739eb3c851188b29f50ed6c0b2194e # v2
   - actions/setup-python@v2 → @e9aba2c848f5ebd159c070c61ea2c4e2b122355e # v2
   - softprops/action-gh-release@v1 → @de2c0eb89ae2a093876385947365aca7b0e5f844 # v1

2. Fixed all script injection issues by moving ${{ }} expressions from run: blocks to env: blocks:
   - Get versions: inputs.pypi-repository-url → PYPI_REPOSITORY_URL, inputs.allow-first-release → ALLOW_FIRST_RELEASE (Python script reads via os.environ)
   - Test sdist: inputs.test-script → TEST_SCRIPT, github.workspace → WORKSPACE
   - Test bdist_wheel: same as Test sdist
   - Logging: step outputs moved to NEW_VERSION, SDIST_RESULT, BDIST_RESULT env vars
   - Push to PyPI: inputs.pypi-repository-url → PYPI_REPOSITORY_URL (used in if-test and assignment)
   - Tag: inputs.github-user → GITHUB_USER, inputs.github-token → GITHUB_TOKEN_INPUT, github.repository → GITHUB_REPOSITORY, steps.get-versions.outputs.tag → GIT_TAG

All remaining ${{ }} expressions are in safe locations (env:, with:, name:, if: fields) — none remain in run: shell scripts.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted `$REPO_URL` expansion in the 'Push to PyPI' step's `twine upload` command by adding double quotes: `--repository-url "$REPO_URL"`. This prevents shell metacharacters in the caller-controlled `pypi-repository-url` input from being interpreted as shell commands.

### Iteration 3

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all four findings in action.yml:

1. Script injection (Get versions): Replaced shell=True subprocess with list-based args (shell=False) to prevent PYPI_REPOSITORY_URL metacharacter injection. Added regex validation of package name and version.

2. Script injection (Test sdist): Replaced `bash -c "$TEST_SCRIPT"` with writing TEST_SCRIPT to a temp file via `printf '%s' "$TEST_SCRIPT" > file` and executing `bash file`.

3. Script injection (Test bdist_wheel): Same fix as Test sdist.

4. GitHub env injection (Get versions): Replaced shell-based `echo name={name} >> $GITHUB_OUTPUT` subprocess calls with direct Python file I/O (`open(github_output, 'a') as f: f.write(...)`), eliminating both the shell injection vector and the newline injection risk. Regex validation of name/version provides an additional layer of protection.

