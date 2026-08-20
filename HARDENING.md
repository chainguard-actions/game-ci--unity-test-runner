<!-- markdownlint-disable -->

# Hardening Report: game-ci--unity-test-runner/v4.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **game-ci--unity-test-runner/v4.2.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: The 'Rewrite ProjectSettings' step in the 'testRunnerInStandaloneWithIL2CPP' job directly interpolates the GitHub Actions expression `${{ matrix.projectPath }}` inside a `run:` shell command. The offending line is: `sed -i "{s/$DefineOriginal/$DefineReplace/g}" ${{ matrix.projectPath }}/ProjectSettings/ProjectSettings.asset`. A matrix value is workflow-controllable and its direct interpolation into a shell command allows script injection. The value should be passed via an env: variable and quoted.

Locations:

- `.github/workflows/main.yml:248`

### unpinned-uses (severity: high)

Multiple `uses:` references in workflow files use mutable tags or version strings instead of pinned 40-character commit SHAs, making them vulnerable to supply-chain attacks. Failing references include: `actions/checkout@v4`, `actions/setup-node@v4`, `actions/cache@v4`, `actions/upload-artifact@v4` (main.yml); `ruairidhwm/action-cats@1.0.2` (cats.yml); `Actions-R-Us/actions-tagger@v2` (versioning.yml).

Locations:

- `.github/workflows/main.yml:19`
- `.github/workflows/cats.yml:13`
- `.github/workflows/versioning.yml:10`

### missing-permissions (severity: medium)

None of the workflow files define a top-level `permissions:` block, and no job within them defines job-level `permissions:` either. This means workflows run with the default (broad) token permissions. This is especially concerning for cats.yml which uses the `pull_request_target` trigger — a high-privilege event that runs in the context of the base repository and can access secrets.

Locations:

- `.github/workflows/main.yml:1`
- `.github/workflows/cats.yml:1`
- `.github/workflows/versioning.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across .github/workflows/main.yml, cats.yml, and versioning.yml:

1. script-injection (main.yml line 248): Moved `${{ matrix.projectPath }}` out of the `run:` shell command into an `env:` block as `PROJECT_PATH`, then referenced it as `"$PROJECT_PATH"` (double-quoted) in the sed command.

2. unpinned-uses: Pinned all action references to full 40-char SHAs:
   - actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262
   - actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020
   - actions/cache@v4 → @0057852bfaa89a56745cba8c7296529d2fc39830
   - actions/upload-artifact@v4 → @ea165f8d65b6e75b540449e92b4886f43607fa02
   - ruairidhwm/action-cats@1.0.2 → @309530f7a011f12e0359514d2c1bb9cef114dd41
   - Actions-R-Us/actions-tagger@v2 → @330ddfac760021349fef7ff62b372f2f691c20fb

3. missing-permissions: Added `permissions: {}` at the top level of all three workflow files. For cats.yml (pull_request_target trigger), added job-level `permissions: pull-requests: write` for the job that needs to post comments. For versioning.yml, added job-level `permissions: contents: write` for the tagging job.

