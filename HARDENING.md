<!-- markdownlint-disable -->

# Hardening Report: patrick-kidger--action_update_python_project/v8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **patrick-kidger--action_update_python_project/v8** was hardened automatically. 10 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple ${{ }} expressions are interpolated directly inside run: shell command strings throughout action.yml, violating sub-rule (a). The most critical instance is `${{ inputs.test-script }}` passed directly to `bash -c "..."` in both the 'Test sdist' and 'Test bdist_wheel' steps — this allows a caller to inject arbitrary shell commands. Additional direct interpolations include: `${{ inputs.pypi-repository-url }}` used unquoted in an `if [ ... ]` comparison and an `export` assignment (sub-rules a and b); `${{ inputs.github-user }}`, `${{ inputs.github-token }}`, and `${{ github.repository }}` interpolated directly into a `git push` URL; `${{ github.workspace }}` interpolated unquoted in pip install and cd commands; and `${{ steps.*.outputs.* }}` values interpolated directly into echo commands in the Logging step.

Locations:

- `action.yml:75`
- `action.yml:80`
- `action.yml:99`
- `action.yml:101`
- `action.yml:104`
- `action.yml:112`
- `action.yml:114`
- `action.yml:117`
- `action.yml:123`
- `action.yml:124`
- `action.yml:125`
- `action.yml:139`
- `action.yml:141`
- `action.yml:147`
- `action.yml:148`

### unpinned-uses (severity: high)

Three uses: references are pinned to mutable version tags instead of immutable 40-character SHA digests, making the action vulnerable to supply-chain attacks if those tags are moved: `actions/checkout@v2` (line 42), `actions/setup-python@v2` (line 45), and `softprops/action-gh-release@v1` (line 155). All three should be pinned to their full commit SHA.

Locations:

- `action.yml:42`
- `action.yml:45`
- `action.yml:155`

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

1. **unpinned-uses**: Pinned all three mutable tag references to full 40-character commit SHAs:
   - actions/checkout@v2 → @0717577d45739eb3c851188b29f50ed6c0b2194e # v2
   - actions/setup-python@v2 → @e9aba2c848f5ebd159c070c61ea2c4e2b122355e # v2
   - softprops/action-gh-release@v1 → @de2c0eb89ae2a093876385947365aca7b0e5f844 # v1

2. **script-injection / static-inline-injection**: Moved all ${{ }} expressions out of run: shell strings into env: maps for every affected step:
   - Get versions: PYPI_REPOSITORY_URL (${{ inputs.pypi-repository-url }}) and ALLOW_FIRST_RELEASE (${{ inputs.allow-first-release }}); Python code updated to use os.environ instead
   - Test sdist: GITHUB_WORKSPACE_PATH (${{ github.workspace }}) and TEST_SCRIPT (${{ inputs.test-script }})
   - Test bdist_wheel: GITHUB_WORKSPACE_PATH (${{ github.workspace }}) and TEST_SCRIPT (${{ inputs.test-script }})
   - Logging: NEW_VERSION, SDIST_RESULT, BDIST_RESULT (all step outputs)
   - Push to PyPI: PYPI_REPOSITORY_URL (${{ inputs.pypi-repository-url }}); comparison and assignment now use the env var
   - Tag: GITHUB_USER, GITHUB_TOKEN_INPUT, GITHUB_REPOSITORY, GIT_TAG; git push URL now uses env vars

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings in hardened/action/action.yml:

1. 'Get versions' step: Changed subprocess.run() from shell=True with an f-string that interpolated repo_url (from inputs.pypi-repository-url) to shell=False with a list of arguments. This ensures the URL value is passed as a literal argument to pip and cannot inject shell commands via metacharacters.

2. 'Test sdist' and 'Test bdist_wheel' steps: Replaced `bash -c "$TEST_SCRIPT"` with writing the script content to a temp file via `printf '%s' "$TEST_SCRIPT" > "$_script_file"` and executing `bash "$_script_file"`. This prevents the test-script content from being parsed as a shell -c argument string, while still allowing the script to run normally as a bash script file.

