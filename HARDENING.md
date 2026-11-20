<!-- markdownlint-disable -->

# Hardening Report: patrick-kidger--action_update_python_project/v8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **patrick-kidger--action_update_python_project/v8** was hardened automatically. 17 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml has composite steps whose uses: refs point at mutable tags instead of full commit SHAs: 'uses: actions/checkout@v2' (line 41), 'uses: actions/setup-python@v2' (line 45), 'uses: softprops/action-gh-release@v1' (line 176).

Locations:

- `action.yml:41`
- `action.yml:45`
- `action.yml:176`

### script-injection (severity: high)

Step 'Get versions', rule (a): expressions are placed straight into the inline python -c script: "repo_url = '${{ inputs.pypi-repository-url }}'.removesuffix('/')" (line 72) and "'${{ inputs.allow-first-release }}' == 'true'" (line 81). A quote in the input breaks out of the Python string or the bash double-quoted string. The repo_url value is then passed to subprocess.run(..., shell=True).

Locations:

- `action.yml:72`
- `action.yml:81`

### script-injection (severity: high)

Step 'Test sdist', rule (a): expressions are interpolated into run:. These are 'python -m uv pip install ${{ github.workspace }}/dist/*.tar.gz' (line 103), '${{ inputs.test-script }}' inside bash -c "..." (line 106) and 'cd ${{ github.workspace }}' (line 115).

Locations:

- `action.yml:103`
- `action.yml:106`
- `action.yml:115`

### script-injection (severity: high)

Step 'Test bdist_wheel', rule (a): expressions are interpolated into run:. These are 'python -m uv pip install ${{ github.workspace }}/dist/*.whl' (line 124), '${{ inputs.test-script }}' inside bash -c "..." (line 127) and 'cd ${{ github.workspace }}' (line 136).

Locations:

- `action.yml:124`
- `action.yml:127`
- `action.yml:136`

### script-injection (severity: high)

Step 'Logging', rule (a): step outputs are interpolated unquoted into run:. These are 'echo new-version=${{ steps.get-versions.outputs.new-version }}' (line 141), 'echo sdist-result=${{ steps.test-sdist.outputs.result }}' (line 142) and 'echo bdist-result=${{ steps.test-bdist-wheel.outputs.result }}' (line 143).

Locations:

- `action.yml:141`
- `action.yml:142`
- `action.yml:143`

### script-injection (severity: high)

Step 'Push to PyPI', rule (a): the input is interpolated unquoted. The lines are 'if [ ${{ inputs.pypi-repository-url }} == https://pypi.org/ ]; then' (line 158) and 'export REPO_URL=${{ inputs.pypi-repository-url }}' (line 161). Rule (b) is also broken: 'python -m twine upload --repository-url $REPO_URL ...' (line 163) expands workflow-controlled data without quotes.

Locations:

- `action.yml:158`
- `action.yml:161`
- `action.yml:163`

### script-injection (severity: high)

Step 'Tag', rule (a): expressions are interpolated into run:. These are 'git tag "${{ steps.get-versions.outputs.tag }}" -m ""' (line 171) and 'git push https://${{ inputs.github-user }}:${{ inputs.github-token }}@github.com/${{ github.repository }} "${{ steps.get-versions.outputs.tag }}"' (line 172).

Locations:

- `action.yml:171`
- `action.yml:172`

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

### github-env-injection (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The 'Get versions' step writes values to `$GITHUB_OUTPUT` via Python `subprocess.run(f'echo name={name} >> $GITHUB_OUTPUT', shell=True)` and similar calls (lines ~87–89). The values `name` and `checkout_version` are read from `pyproject.toml` without sanitization — a newline character in either value would allow injecting additional key=value pairs into GITHUB_OUTPUT (header injection). No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before any of these writes. Additionally, the 'Test sdist' and 'Test bdist_wheel' steps write literal `result=true` / `result=false` to `$GITHUB_OUTPUT` (safe), but the 'Logging' step echoes `${{ steps.get-versions.outputs.new-version }}` etc. directly to stdout without writing to GITHUB_OUTPUT (not a direct injection, but confirms unsanitized data flows).

Locations:

- `action.yml:87`
- `action.yml:88`
- `action.yml:89`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 1fb475a..ae5f2c6 100644
--- a/action.yml
+++ b/action.yml
@@ -2,186 +2,214 @@ name: 'Update Python project: Check version / Test / git tag / GitHub Release /
 author: 'Patrick Kidger'
 description: 'Builds a Python projects, runs tests, and if successful tags and uploads to PyPI'
 inputs:
-    # Common inputs (without defaults)
-    python-version:
-        description: 'The version of Python to use. Set to `false` to not install any Python and instead use a preinstalled one.'
-        required: true
-    pypi-token:
-        description: 'A PyPI password or authentication token for your project. Set to `false` to use some other already-configured authentication mechanism.'
-        required: true
-    github-user:
-        description: 'Your username on GitHub.'
-        required: true
-    github-token:
-        description: 'A GitHub authentication token for your project.'
-        required: true
-    test-script:
-        description: 'A bash script for the tests. A uv venv will be available to install into, using `uv` directly on the PATH, e.g. commands often look like `uv sync --foo`.'
-        required: true
-    # Uncommon inputs (with defaults)
-    pypi-user:
-        description: 'Your username on PyPI.'
-        default: '__token__'
-    pypi-repository-url:
-        description: 'The repository URL to use for PyPI, if different than the default. Passed to `twine upload --repository-url`'
-        default: 'https://pypi.org/'
-    checkout:
-        description: 'Whether to check out the code. Set to `false` to disable this, e.g. if you have already done this.'
-        default: true
-    allow-first-release:
-        description: 'Whether to allow doing a first release to PyPI for a package that does not already exist there. Set to `true` to allow this.'
-        default: false
+  # Common inputs (without defaults)
+  python-version:
+    description: 'The version of Python to use. Set to `false` to not install any Python and instead use a preinstalled one.'
+    required: true
+  pypi-token:
+    description: 'A PyPI password or authentication token for your project. Set to `false` to use some other already-configured authentication mechanism.'
+    required: true
+  github-user:
+    description: 'Your username on GitHub.'
+    required: true
+  github-token:
+    description: 'A GitHub authentication token for your project.'
+    required: true
+  test-script:
+    description: 'A bash script for the tests. A uv venv will be available to install into, using `uv` directly on the PATH, e.g. commands often look like `uv sync --foo`.'
+    required: true
+  # Uncommon inputs (with defaults)
+  pypi-user:
+    description: 'Your username on PyPI.'
+    default: '__token__'
+  pypi-repository-url:
+    description: 'The repository URL to use for PyPI, if different than the default. Passed to `twine upload --repository-url`'
+    default: 'https://pypi.org/'
+  checkout:
+    description: 'Whether to check out the code. Set to `false` to disable this, e.g. if you have already done this.'
+    default: true
+  allow-first-release:
+    description: 'Whether to allow doing a first release to PyPI for a package that does not already exist there. Set to `true` to allow this.'
+    default: false
 branding:
-    icon: 'wind'
-    color: 'yellow'
+  icon: 'wind'
+  color: 'yellow'
 runs:
-    using: "composite"
-    steps:
-        - name: Checkout
           uses: actions/checkout@v2
-          if: inputs.checkout != 'false'
+  using: "composite"
+  steps:
+    - name: Checkout
+      if: inputs.checkout != 'false'
+    - name: Set up Python ${{ inputs.python-version }}
+      uses: chainguard-actions/actions-setup-python@c1869dcf3733fc23f862a7b6ebb625aba29b87e2 # v2
+      if: inputs.python-version != 'false'
+      with:
+        python-version: ${{ inputs.python-version }}
+    - name: Setup
+      shell: bash
+      run: |
+        python -m pip install --upgrade pip
+        python -m pip install --upgrade build uv packaging
+        python -m build
+    - name: Get ver
... (truncated)
```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, static-inline-injection, github-env-injection, prior-fix

**Notes:**

All fixes are in action.yml. Nothing has been run or tested.

- **unpinned-uses:** actions/checkout v2, actions/setup-python v2 and softprops/action-gh-release v1 are now pinned to full commit SHAs from lookup_action_sha, with the tag kept as a comment. The previous harden used chainguard-actions mirror repos instead; this pass pins the upstream repos.
- **Get versions step:** both inputs (pypi-repository-url, allow-first-release) now arrive through env: and Python reads them with os.environ. The pip index lookup is now an argument list instead of shell=True, so the repo URL and package name never reach a shell. The step outputs are written straight to the GITHUB_OUTPUT file with CR/LF removed, which closes the github-env-injection finding.
- **Test sdist / Test bdist_wheel steps:** test-script comes in through env and runs as `bash -c "$TEST_SCRIPT"`. That is deliberately without -e, because upstream ran it with `bash -c` after `set +e`, so a failing line didn't stop the script. github.workspace is replaced with a quoted "$GITHUB_WORKSPACE".
- **Logging, Push to PyPI and Tag steps:** step outputs and inputs are moved to env: and quoted where they're used. That covers REPO_URL in the twine call, and the git push URL built from GH_USER, GH_TOKEN_INPUT and GITHUB_REPOSITORY.
- **Unchanged:** the GitHub release step's with: values. They are action inputs, not shell code, so they aren't an injection risk.

