<!-- markdownlint-disable -->

# Hardening Report: game-ci--unity-test-runner/v4.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **game-ci--unity-test-runner/v4.4.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference external actions using mutable tags or version strings instead of full 40-character SHA commit hashes. This exposes the workflow to supply-chain attacks if the referenced tag is moved or the repository is compromised.

Failing references:
- cats.yml: `uses: ruairidhwm/action-cats@1.0.2`
- main.yml: `uses: actions/checkout@v4`, `uses: actions/setup-node@v4`, `uses: actions/cache@v4`, `uses: actions/upload-artifact@v4`
- versioning.yml: `uses: Actions-R-Us/actions-tagger@v2`

Locations:

- `.github/workflows/cats.yml:14`
- `.github/workflows/main.yml:31`
- `.github/workflows/versioning.yml:11`

### script-injection (severity: high)

Sub-rule (a): The 'Rewrite ProjectSettings' step in main.yml directly interpolates `${{ matrix.projectPath }}` inside a bash `run:` block. The expression is substituted by the GitHub Actions template engine before the shell ever sees it, allowing an attacker who controls the matrix value to inject arbitrary shell commands.

Offending line:
  `sed -i "{s/$DefineOriginal/$DefineReplace/g}" ${{ matrix.projectPath }}/ProjectSettings/ProjectSettings.asset`

Fix: move the value into an env var and quote it: `env: PROJECT_PATH: ${{ matrix.projectPath }}` then use `"$PROJECT_PATH"` in the script.

Locations:

- `.github/workflows/main.yml:453`

### missing-permissions (severity: medium)

None of the three workflow files declare a top-level `permissions:` key, and no individual job within them declares job-level permissions either. Without explicit permissions, workflows inherit the default repository token permissions (which may be broad write-all depending on repository settings), violating the principle of least privilege.

Affected files: cats.yml, main.yml, versioning.yml.

Locations:

- `.github/workflows/cats.yml:1`
- `.github/workflows/main.yml:1`
- `.github/workflows/versioning.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings across cats.yml, main.yml, and versioning.yml:

1. unpinned-uses: Pinned all 6 external action references to full 40-char SHAs (ruairidhwm/action-cats, actions/checkout x14, actions/setup-node, actions/cache x8, actions/upload-artifact x22, Actions-R-Us/actions-tagger). Original tags preserved in comments.

2. script-injection: In main.yml's 'Rewrite ProjectSettings' step, moved `${{ matrix.projectPath }}` out of the bash run block into an `env: PROJECT_PATH:` declaration, then used `"$PROJECT_PATH"` in the sed command.

3. missing-permissions: Added `permissions: {}` at the top level of all three workflow files. Added job-level `permissions: pull-requests: write` to cats.yml (needed for the action to post PR comments) and `permissions: contents: write` to versioning.yml (needed to push/update tags).

