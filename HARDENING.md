<!-- markdownlint-disable -->

# Hardening Report: game-ci--unity-test-runner/v4.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **game-ci--unity-test-runner/v4.3.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: The `testRunnerInStandaloneWithIL2CPP` job in main.yml has a `run:` block that directly interpolates `${{ matrix.projectPath }}` into a shell command: `sed -i "{s/$DefineOriginal/$DefineReplace/g}" ${{ matrix.projectPath }}/ProjectSettings/ProjectSettings.asset`. Any `${{ ... }}` expression inside a `run:` script is a script-injection risk because the value is substituted into the shell command string before the shell parses it.

Locations:

- `.github/workflows/main.yml:246`

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable tags instead of full 40-character SHA commit hashes, making them vulnerable to supply-chain attacks if the referenced tag is moved or overwritten.

- .github/workflows/main.yml: `actions/checkout@v4`, `actions/setup-node@v4`, `actions/cache@v4`, `actions/upload-artifact@v4` (used throughout)
- .github/workflows/cats.yml: `ruairidhwm/action-cats@1.0.2`
- .github/workflows/versioning.yml: `Actions-R-Us/actions-tagger@v2`

Locations:

- `.github/workflows/main.yml:20`
- `.github/workflows/cats.yml:13`
- `.github/workflows/versioning.yml:10`

### missing-permissions (severity: medium)

None of the workflow files define a top-level `permissions:` key, and no individual job defines its own `permissions:` block. Without explicit permissions, workflows run with the default (often broad) token permissions, violating the principle of least privilege. This affects all three workflow files: main.yml, cats.yml, and versioning.yml.

Locations:

- `.github/workflows/main.yml:1`
- `.github/workflows/cats.yml:1`
- `.github/workflows/versioning.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across main.yml, cats.yml, and versioning.yml:

1. **script-injection** (main.yml line 246): Moved `${{ matrix.projectPath }}` out of the `run:` shell script into an `env:` block as `PROJECT_PATH`, then referenced it as `"$PROJECT_PATH"` in the sed command.

2. **unpinned-uses**: Pinned all action references to full SHA hashes:
   - `actions/checkout@v4` → `@11d5960a326750d5838078e36cf38b85af677262`
   - `actions/setup-node@v4` → `@49933ea5288caeca8642d1e84afbd3f7d6820020`
   - `actions/cache@v4` → `@0057852bfaa89a56745cba8c7296529d2fc39830`
   - `actions/upload-artifact@v4` → `@ea165f8d65b6e75b540449e92b4886f43607fa02`
   - `ruairidhwm/action-cats@1.0.2` → `@309530f7a011f12e0359514d2c1bb9cef114dd41`
   - `Actions-R-Us/actions-tagger@v2` → `@330ddfac760021349fef7ff62b372f2f691c20fb`

3. **missing-permissions**: Added `permissions: {}` at the top level of all three workflow files. Added job-level `permissions: pull-requests: write` for cats.yml (needs to post PR comments) and `permissions: contents: write` for versioning.yml (needs to update tags).

