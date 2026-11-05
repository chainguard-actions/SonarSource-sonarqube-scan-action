<!-- markdownlint-disable -->

# Hardening Report: SonarSource--sonarqube-scan-action/v8.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **SonarSource--sonarqube-scan-action/v8.4.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

deprecated-c-cpp/action.yml step 'Configure paths' runs `${GITHUB_ACTION_PATH}/../scripts/configure_paths.sh >> $GITHUB_OUTPUT`. That script echoes values built from untrusted sources straight into GITHUB_OUTPUT: INSTALL_PATH (from inputs.installation-path), SONAR_HOST_URL (an env var inherited from the calling workflow, used for build-wrapper-url), and the SONAR_SCANNER_* values (from steps outputs). None of these go through `printf '%s' ... | tr -d '\n\r'` first, so a newline in any of them could inject extra outputs such as a different download URL.

Locations:

- `deprecated-c-cpp/action.yml:69`
- `scripts/configure_paths.sh:79`

### github-env-injection (severity: high)

deprecated-c-cpp/action.yml step 'Setup action outputs' writes untrusted values to special env files without the `printf '%s' ... | tr -d '\n\r'` sanitization: `echo "SONAR_HOST_URL=${SONAR_HOST_URL}" >> $GITHUB_ENV` passes along an env var inherited from the calling workflow. `echo "${SONAR_SCANNER_BIN_DIR}" >> $GITHUB_PATH`, `echo "${BUILD_WRAPPER_BIN_DIR}" >> $GITHUB_PATH`, `echo "sonar-scanner-binary=${SONAR_SCANNER_BIN}" >> $GITHUB_OUTPUT` and `echo "build-wrapper-binary=${BUILD_WRAPPER_BIN}" >> $GITHUB_OUTPUT` write values derived from steps.configure_paths.outputs.*, which are themselves built from inputs.installation-path.

Locations:

- `deprecated-c-cpp/action.yml:112`
- `deprecated-c-cpp/action.yml:116`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed both github-env-injection findings. Nothing was run or tested.

1. scripts/configure_paths.sh: added a sanitize() helper (printf '%s' | tr -d '\n\r'). It cleans INSTALL_PATH, SONAR_HOST_URL, SONAR_SCANNER_VERSION, SONAR_SCANNER_URL and SONAR_SCANNER_SHA before the script echoes any output lines. That covers what the 'Configure paths' step writes to GITHUB_OUTPUT.

2. deprecated-c-cpp/action.yml, 'Setup action outputs' step: SONAR_HOST_URL and each realpath result now have newlines stripped before they go to GITHUB_ENV, GITHUB_PATH or GITHUB_OUTPUT. Each realpath result is saved to a variable first, so a realpath failure still stops the step under bash -e as it did before. I also put the env-file paths in quotes.

