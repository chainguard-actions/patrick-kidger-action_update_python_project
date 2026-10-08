<!-- markdownlint-disable -->

# Hardening Report: patrick-kidger--action_update_python_project/v7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **patrick-kidger--action_update_python_project/v7** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in action.yml directly interpolate ${{ }} expressions into shell commands, violating rule (a). This allows an attacker who controls the inputs or GitHub context to inject arbitrary shell commands.

1. 'Test sdist' step (line ~76): `${{ github.workspace }}` interpolated unquoted into a pip install command, and `${{ inputs.test-script }}` interpolated directly as the body of `bash -c "..."` — this is arbitrary code execution from a caller-supplied input.
2. 'Test bdist_wheel' step (line ~92): same patterns — `${{ github.workspace }}` and `${{ inputs.test-script }}` directly in shell.
3. 'Logging' step (line ~108): `${{ steps.get-versions.outputs.new-version }}`, `${{ steps.test-sdist.outputs.result }}`, `${{ steps.test-bdist-wheel.outputs.result }}` interpolated directly into echo commands.
4. 'Tag' step (line ~121): `${{ steps.get-versions.outputs.tag }}`, `${{ inputs.github-user }}`, `${{ inputs.github-token }}`, and `${{ github.repository }}` all interpolated directly into git tag and git push commands — credentials and attacker-controlled values embedded in a shell URL.

Locations:

- `action.yml:76`
- `action.yml:80`
- `action.yml:88`
- `action.yml:92`
- `action.yml:96`
- `action.yml:104`
- `action.yml:108`
- `action.yml:121`

### unpinned-uses (severity: high)

Three uses: references in action.yml are pinned to mutable version tags instead of immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved:
- `actions/checkout@v2` (line 28)
- `actions/setup-python@v2` (line 31)
- `softprops/action-gh-release@v1` (line 128)

Locations:

- `action.yml:28`
- `action.yml:31`
- `action.yml:128`

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

1. unpinned-uses: Pinned all three action references to full commit SHAs:
   - actions/checkout@v2 → @0717577d45739eb3c851188b29f50ed6c0b2194e # v2
   - actions/setup-python@v2 → @e9aba2c848f5ebd159c070c61ea2c4e2b122355e # v2
   - softprops/action-gh-release@v1 → @de2c0eb89ae2a093876385947365aca7b0e5f844 # v1

2. script-injection / static-inline-injection: Moved all ${{ }} expressions from run: blocks into env: blocks:
   - Test sdist step: github.workspace → WORKSPACE env var; inputs.test-script → TEST_SCRIPT env var (written to temp file and executed with 'bash -eo pipefail' to preserve errexit semantics)
   - Test bdist_wheel step: Same pattern as Test sdist
   - Logging step: step outputs moved to NEW_VERSION, SDIST_RESULT, BDIST_RESULT env vars
   - Tag step: steps.get-versions.outputs.tag → GIT_TAG, inputs.github-user → GITHUB_USER, inputs.github-token → GITHUB_TOKEN_INPUT, github.repository → GITHUB_REPOSITORY_NAME; all used as shell variables in the git push URL

