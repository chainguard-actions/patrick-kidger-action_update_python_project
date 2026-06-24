<!-- markdownlint-disable -->

# Hardening Report: patrick-kidger--action_update_python_project/v8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **patrick-kidger--action_update_python_project/v8** was hardened automatically. 11 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three action references in action.yml use mutable tag refs instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if those tags are moved: `actions/checkout@v2`, `actions/setup-python@v2`, and `softprops/action-gh-release@v1`.

Locations:

- `action.yml:43`
- `action.yml:46`
- `action.yml:196`

### script-injection (severity: high)

Multiple ${{ }} expressions are interpolated directly inside run: shell blocks (sub-rule a), bypassing shell quoting and enabling script injection:

1. 'Get versions' step (line ~72): `${{ inputs.pypi-repository-url }}` and `${{ inputs.allow-first-release }}` are embedded as Python string literals inside the shell run block. An attacker-controlled value could break out of the Python string context.

2. 'Test sdist' step (line ~84): `${{ inputs.test-script }}` is interpolated directly into `bash -c "${{ inputs.test-script }}"` — a critical arbitrary code execution vector since the entire test script is caller-supplied and injected verbatim into a bash -c invocation.

3. 'Test bdist_wheel' step (line ~100): Same pattern — `${{ inputs.test-script }}` interpolated directly into `bash -c "${{ inputs.test-script }}"`.

4. 'Test sdist' and 'Test bdist_wheel' steps: `${{ github.workspace }}` is interpolated unquoted in pip install and cd commands.

5. 'Logging' step (line ~109): `${{ steps.get-versions.outputs.new-version }}`, `${{ steps.test-sdist.outputs.result }}`, `${{ steps.test-bdist-wheel.outputs.result }}` interpolated directly in echo commands.

6. 'Push to PyPI' step (line ~120): `${{ inputs.pypi-repository-url }}` interpolated unquoted in `if [ ${{ inputs.pypi-repository-url }} == https://pypi.org/ ]` and `export REPO_URL=${{ inputs.pypi-repository-url }}`.

7. 'Tag' step (line ~133): `${{ inputs.github-user }}`, `${{ inputs.github-token }}`, `${{ github.repository }}`, and `${{ steps.get-versions.outputs.tag }}` all interpolated directly in the git push URL.

Locations:

- `action.yml:72`
- `action.yml:84`
- `action.yml:100`
- `action.yml:109`
- `action.yml:120`
- `action.yml:133`

### github-env-injection (severity: high)

The 'Get versions' step writes values to $GITHUB_OUTPUT via Python subprocess.run calls without newline sanitization. The inputs `${{ inputs.pypi-repository-url }}` and `${{ inputs.allow-first-release }}` are interpolated directly into the Python script string that also performs the GITHUB_OUTPUT writes — a malicious input containing newlines could inject additional key=value pairs into $GITHUB_OUTPUT. The `name` and `checkout_version` values written to $GITHUB_OUTPUT come from pyproject.toml but are also written without sanitization (e.g. `echo name={name} >> $GITHUB_OUTPUT`), and a compromised or attacker-influenced pyproject.toml could inject newlines. The downstream `${{ steps.get-versions.outputs.tag }}` value is then used unsanitized in the 'Tag' step's git push command.

Locations:

- `action.yml:72`
- `action.yml:84`
- `action.yml:100`

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

Fixed all security findings in action.yml: (1) Pinned actions/checkout@v2 to SHA ee0669bd1cc54295c223e0bb666b733df41de1c5, actions/setup-python@v2 to SHA e9aba2c848f5ebd159c070c61ea2c4e2b122355e, and softprops/action-gh-release@v1 to SHA de2c0eb89ae2a093876385947365aca7b0e5f844. (2) Moved all ${{ inputs.pypi-repository-url }}, ${{ inputs.allow-first-release }}, ${{ inputs.test-script }}, ${{ github.workspace }}, ${{ inputs.github-user }}, ${{ inputs.github-token }}, ${{ github.repository }}, and ${{ steps.get-versions.outputs.* }} expressions from run: blocks into env: blocks, referencing them as plain environment variables in shell scripts. (3) Replaced subprocess.run shell echo commands for GITHUB_OUTPUT with a Python safe_write() function that strips newlines/carriage returns before writing, preventing newline injection attacks. (4) Fixed the Push to PyPI step to use ${PYPI_REPOSITORY_URL} env var instead of inline ${{ inputs.pypi-repository-url }}. (5) Fixed the Tag step to use env vars for all GitHub context values including credentials.

