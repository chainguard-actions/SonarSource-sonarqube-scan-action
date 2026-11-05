<!-- markdownlint-disable -->

# Hardening Report: SonarSource--sonarqube-scan-action/v8.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **SonarSource--sonarqube-scan-action/v8.3.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Configure paths' step in deprecated-c-cpp/action.yml runs `${GITHUB_ACTION_PATH}/../scripts/configure_paths.sh >> $GITHUB_OUTPUT`. That script echoes values built from untrusted sources straight into GITHUB_OUTPUT: INSTALL_PATH (from `inputs.installation-path`) goes into sonar-scanner-dir, sonar-scanner-bin, build-wrapper-dir and build-wrapper-bin, and the inherited workflow env var SONAR_HOST_URL goes into build-wrapper-url. Nothing removes newlines first (no `printf '%s' ... | tr -d '\n\r'`), so someone who controls either value can inject extra outputs.

Locations:

- `deprecated-c-cpp/action.yml:69`
- `scripts/configure_paths.sh:83`

### github-env-injection (severity: high)

The 'Setup action outputs' step in deprecated-c-cpp/action.yml writes untrusted values to special env files without the `printf '%s' ... | tr -d '\n\r'` sanitization. It writes `echo "SONAR_HOST_URL=${SONAR_HOST_URL}" >> $GITHUB_ENV`, where SONAR_HOST_URL is inherited from the calling workflow. It writes `echo "${SONAR_SCANNER_BIN_DIR}" >> $GITHUB_PATH` and `echo "${BUILD_WRAPPER_BIN_DIR}" >> $GITHUB_PATH`, which come from steps.configure_paths.outputs and so from inputs.installation-path. It also writes sonar-scanner-binary and build-wrapper-binary to $GITHUB_OUTPUT from the same step outputs.

Locations:

- `deprecated-c-cpp/action.yml:112`
- `deprecated-c-cpp/action.yml:116`
- `deprecated-c-cpp/action.yml:120`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

scripts/configure_paths.sh: added an emit_output helper. It strips \n and \r from each value (printf '%s' | tr -d '\n\r') before printing key=value. All 7 outputs now go through it, including the ones built from INSTALL_PATH and SONAR_HOST_URL. deprecated-c-cpp/action.yml 'Setup action outputs': SONAR_HOST_URL is sanitized before it is written to GITHUB_ENV. Each realpath result is captured in its own assignment first, so a realpath failure still stops the step under errexit, and then sanitized before it is written to GITHUB_PATH or GITHUB_OUTPUT. The redirect targets are now quoted. Nothing was run to test these changes.

