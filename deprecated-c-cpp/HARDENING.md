<!-- markdownlint-disable -->

# Hardening Report: SonarSource--sonarqube-scan-action--deprecated-c-cpp/v8.5.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **SonarSource--sonarqube-scan-action--deprecated-c-cpp/v8.5.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Setup action outputs' step (a composite action step) writes untrusted values to special environment files without the `printf '%s' ... | tr -d '\n\r'` sanitization step. (1) Line 116: `echo "SONAR_HOST_URL=${SONAR_HOST_URL}" >> $GITHUB_ENV`. SONAR_HOST_URL is an env var inherited from the calling workflow, and it is forwarded to GITHUB_ENV unchanged (case e). (2) Line 120: `echo "${SONAR_SCANNER_BIN_DIR}" >> $GITHUB_PATH` and line 124: `echo "sonar-scanner-binary=${SONAR_SCANNER_BIN}" >> $GITHUB_OUTPUT`. (3) The matching lines for BUILD_WRAPPER_BIN_DIR (GITHUB_PATH) and BUILD_WRAPPER_BIN (GITHUB_OUTPUT). These values come from steps.configure_paths.outputs.*, which are derived from inputs.installation-path, passed through realpath, and written without stripping newlines. A newline in any of these values could inject extra environment variables, PATH entries or outputs.

Locations:

- `action.yml:116`
- `action.yml:120`
- `action.yml:124`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

In action.yml, the 'Setup action outputs' step now removes newline and carriage-return characters before writing to the GitHub env, path and output files. This covers SONAR_HOST_URL (GITHUB_ENV), the sonar-scanner and build-wrapper bin directories (GITHUB_PATH), and the sonar-scanner-binary and build-wrapper-binary outputs (GITHUB_OUTPUT). The realpath call still runs as its own assignment and the cleanup happens on the next line, so if realpath fails the step still stops. The env-file paths are now quoted too. Not tested; no other files changed.

