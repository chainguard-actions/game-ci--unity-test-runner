<!-- markdownlint-disable -->

# Hardening Report: game-ci--unity-test-runner/v5.0.0-beta.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **game-ci--unity-test-runner/v5.0.0-beta.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references across the workflow files use mutable tags or version strings instead of pinned 40-character commit SHAs, making the workflows vulnerable to supply-chain attacks if any referenced action is compromised or its tag is moved.

Failing references in .github/workflows/cats.yml:
- `uses: ruairidhwm/action-cats@1.0.2`

Failing references in .github/workflows/versioning.yml:
- `uses: Actions-R-Us/actions-tagger@v2`

Failing references in .github/workflows/main.yml:
- `uses: actions/checkout@v4` (multiple occurrences)
- `uses: actions/setup-node@v4`
- `uses: actions/cache@v4` (multiple occurrences)
- `uses: actions/upload-artifact@v4` (multiple occurrences)

Locations:

- `.github/workflows/cats.yml:13`
- `.github/workflows/versioning.yml:10`
- `.github/workflows/main.yml:22`

### missing-permissions (severity: medium)

None of the workflow files define a `permissions:` key at the top level, and no job within them defines job-level permissions either. This means workflows run with the default (potentially broad) token permissions. All three workflow files are affected: cats.yml, versioning.yml, and main.yml.

Locations:

- `.github/workflows/cats.yml:1`
- `.github/workflows/versioning.yml:1`
- `.github/workflows/main.yml:1`

### script-injection (severity: high)

Sub-rule (a): A `${{ }}` expression is interpolated directly inside a `run:` shell command string. In the `testRunnerInStandaloneWithIL2CPP` job, the `Rewrite ProjectSettings` step passes `${{ matrix.projectPath }}` directly into a `sed` command without routing through an env var:

```
sed -i "{s/$DefineOriginal/$DefineReplace/g}" ${{ matrix.projectPath }}/ProjectSettings/ProjectSettings.asset
```

The `matrix.projectPath` value is substituted by the GitHub Actions template engine before the shell ever sees the command, allowing shell metacharacters in the value to be interpreted by bash.

Locations:

- `.github/workflows/main.yml:441`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings:
1. unpinned-uses: Pinned all action references to full commit SHAs: ruairidhwm/action-cats@309530f (1.0.2), Actions-R-Us/actions-tagger@330ddfa (v2), actions/checkout@11d5960 (v4), actions/setup-node@49933ea (v4), actions/cache@0057852 (v4), actions/upload-artifact@ea165f8 (v4). All 20+ occurrences across cats.yml, versioning.yml, and main.yml were updated.
2. missing-permissions: Added `permissions: {}` at the top level of all three workflow files. cats.yml job gets `pull-requests: write`, versioning.yml job gets `contents: write`.
3. script-injection: Moved `${{ matrix.projectPath }}` in the `Rewrite ProjectSettings` step into an `env:` block as `PROJECT_PATH`, and updated the sed command to use `"$PROJECT_PATH"` instead of the direct template expression.

