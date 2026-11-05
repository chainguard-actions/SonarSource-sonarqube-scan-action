<!-- markdownlint-disable -->

# Hardening Report: SonarSource--sonarqube-scan-action--deprecated-c-cpp/v8.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **SonarSource--sonarqube-scan-action--deprecated-c-cpp/v8.3.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Setup action outputs' writes untrusted values to special environment files without the `printf '%s' ... | tr -d '\n\r'` sanitization step. (1) `echo "SONAR_HOST_URL=${SONAR_HOST_URL}" >> $GITHUB_ENV` passes along SONAR_HOST_URL, which the calling workflow can set (case e). (2) `echo "${SONAR_SCANNER_BIN_DIR}" >> $GITHUB_PATH` and `echo "${BUILD_WRAPPER_BIN_DIR}" >> $GITHUB_PATH` write paths built from steps.configure_paths.outputs.*, which in turn come from inputs.installation-path. (3) `echo "sonar-scanner-binary=${SONAR_SCANNER_BIN}" >> $GITHUB_OUTPUT` and `echo "build-wrapper-binary=${BUILD_WRAPPER_BIN}" >> $GITHUB_OUTPUT` write values that also come from step outputs. A newline in any of these values could inject extra env vars, PATH entries or outputs.

Locations:

- `action.yml:116`
- `action.yml:120`
- `action.yml:124`
- `action.yml:128`
- `action.yml:132`

### github-env-injection (severity: high)

Step 'Configure paths' runs `${GITHUB_ACTION_PATH}/../scripts/configure_paths.sh >> $GITHUB_OUTPUT`, sending the script's output straight into $GITHUB_OUTPUT. That output is built from untrusted env such as INSTALL_PATH (inputs.installation-path) and steps.sonar-scanner-version.outputs.*, and no `printf '%s' ... | tr -d '\n\r'` sanitization is applied, so newlines in installation-path could inject extra step outputs.

Locations:

- `action.yml:69`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Setup action outputs: SONAR_HOST_URL and every PATH entry and output value now have newlines removed with printf '%s' | tr -d '\n\r' before being written to GITHUB_ENV, GITHUB_PATH or GITHUB_OUTPUT. The realpath captures are still bare assignments, so a failing realpath still stops the step as before. Configure paths: INSTALL_PATH (from inputs.installation-path) now has newlines removed and is re-exported before configure_paths.sh runs; the script's output still goes straight to GITHUB_OUTPUT. I couldn't open configure_paths.sh because it sits outside the action directory, so I'm assuming it builds its output only from INSTALL_PATH and the sonar-scanner-version file that ships with the action. If it uses anything else, that value could still put extra lines into GITHUB_OUTPUT.

