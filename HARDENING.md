<!-- markdownlint-disable -->

# Hardening Report: SonarSource--sonarqube-scan-action/v8.4.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **SonarSource--sonarqube-scan-action/v8.4.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Configure paths' runs `${GITHUB_ACTION_PATH}/../scripts/configure_paths.sh >> $GITHUB_OUTPUT`. The script echoes values built from INSTALL_PATH (from inputs.installation-path), the inherited SONAR_HOST_URL env var (build-wrapper-url), and earlier step outputs (sonar-scanner-url/sha), for example `echo "sonar-scanner-dir=${INSTALL_PATH}/..."` and `echo "build-wrapper-url=${SONAR_HOST_URL%/}/static/..."`. None of these values goes through `printf '%s' ... | tr -d '\n\r'` first, so a newline in any of them can inject extra step outputs.

Locations:

- `deprecated-c-cpp/action.yml:69`
- `scripts/configure_paths.sh:82`

### github-env-injection (severity: high)

Step 'Setup action outputs' writes untrusted values to the special env files without sanitization. `echo "SONAR_HOST_URL=${SONAR_HOST_URL}" >> $GITHUB_ENV` passes on an env var inherited from the calling workflow. The step also writes paths derived from step outputs and inputs.installation-path: SONAR_SCANNER_BIN_DIR and BUILD_WRAPPER_BIN_DIR go to $GITHUB_PATH, and sonar-scanner-binary and build-wrapper-binary go to $GITHUB_OUTPUT. None of these writes is preceded by `printf '%s' ... | tr -d '\n\r'`.

Locations:

- `deprecated-c-cpp/action.yml:116`
- `deprecated-c-cpp/action.yml:120`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

scripts/configure_paths.sh: added a write_output helper that removes newlines and carriage returns from each value (printf '%s' | tr -d '\n\r') and then prints key=value. All eight output lines now go through it. deprecated-c-cpp/action.yml 'Setup action outputs': SONAR_HOST_URL (to GITHUB_ENV), both bin dirs (to GITHUB_PATH) and both binary paths (to GITHUB_OUTPUT) have newlines removed before they are written. Each realpath call is still its own assignment, so a failing realpath still stops the step. The file redirections are now quoted. Nothing was run to test these changes.

