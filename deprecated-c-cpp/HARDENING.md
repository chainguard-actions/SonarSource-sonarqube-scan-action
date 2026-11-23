<!-- markdownlint-disable -->

# Hardening Report: SonarSource--sonarqube-scan-action--deprecated-c-cpp/v8.5.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **SonarSource--sonarqube-scan-action--deprecated-c-cpp/v8.5.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Setup action outputs' step writes workflow-controlled values to the runner's special files without the required `printf '%s' ... | tr -d '\n\r'` sanitization. (1) It forwards the inherited process env var SONAR_HOST_URL, which the calling workflow sets: `echo "SONAR_HOST_URL=${SONAR_HOST_URL}" >> $GITHUB_ENV`. (2) It writes paths derived from steps.configure_paths.outputs.* (which come from inputs.installation-path) to $GITHUB_PATH and $GITHUB_OUTPUT: `echo "${SONAR_SCANNER_BIN_DIR}" >> $GITHUB_PATH`, `echo "sonar-scanner-binary=${SONAR_SCANNER_BIN}" >> $GITHUB_OUTPUT`, `echo "${BUILD_WRAPPER_BIN_DIR}" >> $GITHUB_PATH`, `echo "build-wrapper-binary=${BUILD_WRAPPER_BIN}" >> $GITHUB_OUTPUT`. A value containing a newline could inject extra environment variables, PATH entries or outputs.

Locations:

- `action.yml:116`
- `action.yml:119`
- `action.yml:123`
- `action.yml:127`
- `action.yml:131`

### github-env-injection (severity: high)

The 'Configure paths' step sends the output of configure_paths.sh straight to $GITHUB_OUTPUT with no sanitization: `run: ${GITHUB_ACTION_PATH}/../scripts/configure_paths.sh >> $GITHUB_OUTPUT`. That output is built from workflow-controlled data: INSTALL_PATH (inputs.installation-path) and steps.sonar-scanner-version.outputs.*. The value is never passed through `printf '%s' ... | tr -d '\n\r'` before the write.

Locations:

- `action.yml:69`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Both github-env-injection findings are fixed in action.yml.

- **'Configure paths' step:** every workflow-controlled env value (INSTALL_PATH and the scanner version, URLs and checksums) now has newlines stripped with printf | tr -d '\n\r' before configure_paths.sh runs. The script's output is captured with a bare assignment, so the step still fails if the script fails, as upstream did. Each line of output is then cleaned and written to $GITHUB_OUTPUT on its own. Blank lines are skipped and the key=value lines are kept.

- **'Setup action outputs' step:** SONAR_HOST_URL and each realpath-derived path are cleaned with printf | tr -d '\n\r' before being written to $GITHUB_ENV, $GITHUB_PATH or $GITHUB_OUTPUT. Each realpath result goes into a plain variable first, so a realpath failure still stops the step. Every redirect target is now quoted.

Nothing has been run or tested. The action folder only contains action.yml (the ../scripts directory isn't in this checkout), so I couldn't read configure_paths.sh. The line-by-line handling assumes that script prints one key=value per line. If it uses the multi-line name<<EOF form, the per-line rewrite would still write it out unchanged, but the newline stripping on its inputs would affect those values.

