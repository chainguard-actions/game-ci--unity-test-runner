<!-- markdownlint-disable -->

# Hardening Report: game-ci--unity-test-runner/v4.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **game-ci--unity-test-runner/v4.3.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Rewrite ProjectSettings' step in the 'testRunnerInStandaloneWithIL2CPP' job directly interpolates the expression `${{ matrix.projectPath }}` inside a `run:` shell command. This allows a workflow-controllable matrix value to be injected into the shell command before the shell ever sees it. The offending line is: `sed -i "{s/$DefineOriginal/$DefineReplace/g}" ${{ matrix.projectPath }}/ProjectSettings/ProjectSettings.asset`. The fix is to move the value into an env var and quote it: `env: PROJECT_PATH: ${{ matrix.projectPath }}` then use `"$PROJECT_PATH"` in the script.

Locations:

- `.github/workflows/main.yml:270`

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags instead of pinned full 40-character SHA digests, making them vulnerable to supply-chain attacks if the tag is moved. Failing references in main.yml: `actions/checkout@v4`, `actions/setup-node@v4`, `actions/cache@v4`, `actions/upload-artifact@v4` (each used many times). Failing reference in cats.yml: `ruairidhwm/action-cats@1.0.2`. Failing reference in versioning.yml: `Actions-R-Us/actions-tagger@v2`.

Locations:

- `.github/workflows/main.yml:20`
- `.github/workflows/cats.yml:13`
- `.github/workflows/versioning.yml:10`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` key, and no individual job within them defines a `permissions:` key either. Without explicit permissions, workflows run with the default (potentially broad) token permissions. All three files — main.yml, cats.yml, and versioning.yml — are affected. Note: cats.yml uses the `pull_request_target` trigger, which runs with write permissions by default, making the missing explicit permissions restriction especially important there.

Locations:

- `.github/workflows/main.yml:1`
- `.github/workflows/cats.yml:1`
- `.github/workflows/versioning.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across main.yml, cats.yml, and versioning.yml:

1. script-injection (main.yml line 270): Moved `${{ matrix.projectPath }}` from the `run:` shell command into an `env:` block as `PROJECT_PATH`, then used `"$PROJECT_PATH"` safely in the sed command.

2. unpinned-uses: Pinned all mutable tag references to full 40-char SHAs — actions/checkout@v4 (15x), actions/setup-node@v4 (1x), actions/cache@v4 (7x), actions/upload-artifact@v4 (22x) in main.yml; ruairidhwm/action-cats@1.0.2 in cats.yml; Actions-R-Us/actions-tagger@v2 in versioning.yml.

3. missing-permissions: Added top-level `permissions:` blocks — main.yml gets `contents: read`; cats.yml (pull_request_target trigger) gets `contents: read, pull-requests: write`; versioning.yml gets `contents: write` (needed for tag management).

