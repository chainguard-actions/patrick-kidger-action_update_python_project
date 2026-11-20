<!-- markdownlint-disable -->

# Hardening Report: patrick-kidger--action_update_python_project/v7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **patrick-kidger--action_update_python_project/v7** was hardened automatically. 11 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml has composite steps whose uses: refs are mutable tags, not 40-character commit SHAs: 'uses: actions/checkout@v2' (line 30), 'uses: actions/setup-python@v2' (line 33), 'uses: softprops/action-gh-release@v1' (line 143).

Locations:

- `action.yml:30`
- `action.yml:33`
- `action.yml:143`

### script-injection (severity: high)

Rule (a), step 'Test sdist': a ${{ }} expression is placed directly in the run: script. 'python -m uv pip install ${{ github.workspace }}/dist/*.tar.gz ...' (line 80), 'bash -c " ${{ inputs.test-script }} "' (line 83) and 'cd ${{ github.workspace }}' (line 92). Line 92 is also unquoted. inputs.test-script is substituted into the shell text before bash runs.

Locations:

- `action.yml:80`
- `action.yml:83`
- `action.yml:92`

### script-injection (severity: high)

Rule (a), step 'Test bdist_wheel': a ${{ }} expression is placed directly in the run: script. 'python -m uv pip install ${{ github.workspace }}/dist/*.whl ...' (line 101), 'bash -c " ${{ inputs.test-script }} "' (line 104) and 'cd ${{ github.workspace }}' (line 113).

Locations:

- `action.yml:101`
- `action.yml:104`
- `action.yml:113`

### script-injection (severity: high)

Rule (a), step 'Logging': unquoted step outputs are placed directly in the run: script. 'echo new-version=${{ steps.get-versions.outputs.new-version }}' (line 118), 'echo sdist-result=${{ steps.test-sdist.outputs.result }}' (line 119) and 'echo bdist-result=${{ steps.test-bdist-wheel.outputs.result }}' (line 120).

Locations:

- `action.yml:118`

### script-injection (severity: high)

Rule (a), step 'Tag': expressions are placed directly in the run: script. 'git tag "${{ steps.get-versions.outputs.tag }}" -m ""' (line 138) and 'git push https://${{ inputs.github-user }}:${{ inputs.github-token }}@github.com/${{ github.repository }} "${{ steps.get-versions.outputs.tag }}"' (line 139). The tag value comes from pyproject.toml in the repository, so it is attacker-controllable.

Locations:

- `action.yml:138`
- `action.yml:139`

### github-env-injection (severity: high)

Step 'Get versions' writes values to $GITHUB_OUTPUT without stripping newlines. The project name and version are read from the checked-out pyproject.toml, which a PR can change. They are written with subprocess.run(f'echo name={name} >> $GITHUB_OUTPUT', shell=True) and f'echo tag=v{checkout_version} >> $GITHUB_OUTPUT' (lines 63-64). No printf '%s' | tr -d '\n\r' step runs first, so extra output keys can be injected. Because shell=True is used, these values are also parsed by a shell.

Locations:

- `action.yml:63`
- `action.yml:64`

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

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index d3f746d..1e8f3bf 100644
--- a/action.yml
+++ b/action.yml
@@ -2,153 +2,182 @@ name: 'Update Python project: Check version / Test / git tag / GitHub Release /
 author: 'Patrick Kidger'
 description: 'Builds a Python projects, runs tests, and if successful tags and uploads to PyPI'
 inputs:
-    python-version:
-        description: 'The version of Python to use.'
-        required: true
-    pypi-user:
-        description: 'Your username on PyPI.'
-        default: '__token__'
-    pypi-token:
-        description: 'A PyPI password or authentication token for your project.'
-        required: true
-    github-user:
-        description: 'Your username on GitHub.'
-        required: true
-    github-token:
-        description: 'A GitHub authentication token for your project.'
-        required: true
-    test-script:
-        description: 'The script for the tests.'
-        required: true
+  python-version:
+    description: 'The version of Python to use.'
+    required: true
+  pypi-user:
+    description: 'Your username on PyPI.'
+    default: '__token__'
+  pypi-token:
+    description: 'A PyPI password or authentication token for your project.'
+    required: true
+  github-user:
+    description: 'Your username on GitHub.'
+    required: true
+  github-token:
+    description: 'A GitHub authentication token for your project.'
+    required: true
+  test-script:
+    description: 'The script for the tests.'
+    required: true
 branding:
-    icon: 'wind'
-    color: 'yellow'
+  icon: 'wind'
+  color: 'yellow'
 runs:
-    using: "composite"
-    steps:
-        - name: Checkout code
           uses: actions/checkout@v2
+  using: "composite"
+  steps:
+    - name: Checkout code
+    - name: Set up Python ${{ inputs.python-version }}
+      uses: chainguard-actions/actions-setup-python@c1869dcf3733fc23f862a7b6ebb625aba29b87e2 # v2
+      with:
+        python-version: ${{ inputs.python-version }}
+    - name: Setup
+      shell: bash
+      run: |
+        python -m pip install --upgrade pip
+        python -m pip install build uv
+        python -m build
+    - name: Get versions
+      id: get-versions
+      shell: bash
+      run: |
+        python -c "
+        import subprocess
+        import tomllib
 
-        - name: Set up Python ${{ inputs.python-version }}
-          uses: actions/setup-python@v2
-          with:
-              python-version: ${{ inputs.python-version }}
+        vparse = lambda x: tuple(map(int, x.split('.')))
 
-        - name: Setup
-          shell: bash
-          run: |
-              python -m pip install --upgrade pip
-              python -m pip install build uv
-              python -m build
+        with open('pyproject.toml', 'rb') as f:
+            data = tomllib.load(f)
+        name = data['project']['name']
+        checkout_version = data['project']['version']
 
-        - name: Get versions
-          id: get-versions
-          shell: bash
-          run: |
-              python -c "
-              import subprocess
-              import tomllib
+        pypi_version = subprocess.run(f'python -m pip index versions {name}',
+                                      shell=True, capture_output=True).stdout
+        pypi_version = pypi_version.split(b'\n', 1)[0].split(b' ')[1][1:-1].decode('utf-8')
+        new_version = str(vparse(checkout_version) > vparse(pypi_version)).lower()
 
-              vparse = lambda x: tuple(map(int, x.split('.')))
+        def safe(val):
+            return val.replace('\n', '').replace('\r', '')
 
-              with open('pyproject.toml', 'rb') as f:
-                  data = tomllib.load(f)
-              name = data['project']['name']
-              checkout_version = data['project']['version']
-
-              pypi_version = subprocess.run(f'python -m pip index versions {name}',
-                                            shell=True, capture_output=True).stdout
-              pypi_version = pypi_version.split(b'\n', 1)[0].split(b' ')[1][1:-1].decode('utf-8')
-              new_version = str(vpars
... (truncated)
```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-inline-injection, prior-fix

**Notes:**

Hardened action.yml. Changes were not run or tested.
- Pinned actions/checkout v2 (0717577d45739eb3c851188b29f50ed6c0b2194e), actions/setup-python v2 (e9aba2c848f5ebd159c070c61ea2c4e2b122355e) and softprops/action-gh-release v1 (de2c0eb89ae2a093876385947365aca7b0e5f844), with the tag kept in a comment. The previous harden used a chainguard-actions mirror for setup-python; this pass uses the upstream repos.
- Test sdist / Test bdist_wheel: github.workspace and inputs.test-script moved to env (WORKSPACE_DIR, TEST_SCRIPT). The test runs as bash -c "$TEST_SCRIPT" after set +e, without -e, matching upstream. cd paths and $GITHUB_OUTPUT are quoted; the dist/* glob stays outside the quotes.
- Logging: step outputs moved to env and echoed quoted.
- Tag: tag, github-user, github-token and repository moved to env; git tag and git push use only quoted variables.
- Get versions: Python writes name/tag/new-version straight to the GITHUB_OUTPUT file through safe(), which strips \n and \r, instead of subprocess shell=True echo. pip index versions now gets a list of arguments without shell=True.
- Left as is: 4-space indentation, and the release step's with: inputs (not run by a shell).
- Behaviour changes: $VARs in the test script are now expanded by the inner bash -c rather than the outer shell, so a script relying on the outer shell's expansion or quoting could act a little differently. A name or version with spaces or shell characters is now written as-is (minus newlines) instead of being mangled by echo.

