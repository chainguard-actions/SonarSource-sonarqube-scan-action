<!-- markdownlint-disable -->

# Hardening Report: SonarSource--sonarqube-scan-action--deprecated-c-cpp/v8.4.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **SonarSource--sonarqube-scan-action--deprecated-c-cpp/v8.4.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Setup action outputs' writes untrusted values to the special env files without the required sanitization (printf '%s' ... | tr -d '\n\r'). (1) `echo "SONAR_HOST_URL=${SONAR_HOST_URL}" >> $GITHUB_ENV`: SONAR_HOST_URL is inherited from the calling workflow's env and is passed on to $GITHUB_ENV unsanitized. A value containing a newline could inject other variables, for example LD_PRELOAD or BASH_ENV. (2) `echo "${SONAR_SCANNER_BIN_DIR}" >> $GITHUB_PATH` and `echo "${BUILD_WRAPPER_BIN_DIR}" >> $GITHUB_PATH` write paths derived from steps.configure_paths.outputs.* (which come from inputs.installation-path) without stripping newlines. (3) The `sonar-scanner-binary=` and `build-wrapper-binary=` lines are written to $GITHUB_OUTPUT the same way. Each value should go through `safe=$(printf '%s' "$VAR" | tr -d '\n\r')` before it is written.

Locations:

- `action.yml:116`
- `action.yml:120`
- `action.yml:124`
- `action.yml:128`
- `action.yml:132`

### github-env-injection (severity: high)

Step 'Configure paths' runs `${GITHUB_ACTION_PATH}/../scripts/configure_paths.sh >> $GITHUB_OUTPUT`. The script's output is built from INSTALL_PATH (inputs.installation-path) and from step outputs passed in through env, and it goes straight into $GITHUB_OUTPUT with no newline sanitization. An installation-path containing a newline could inject arbitrary step outputs, such as a fake sonar-scanner-url or sonar-scanner-sha, which later steps use for downloads.

Locations:

- `action.yml:69`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Two steps in action.yml changed.

'Configure paths': before the script runs, INSTALL_PATH and the sonar-scanner version, URL and SHA env values have their newlines stripped. The script's output is captured first; a failing script still stops the step, because the plain assignment runs under bash -e. Each output line then has CR/LF stripped, and only key=value lines are written to GITHUB_OUTPUT. Any other non-empty lines go to the log instead.

'Setup action outputs': SONAR_HOST_URL, both bin dirs and both binary paths each go through printf '%s' | tr -d '\n\r' before being written to GITHUB_ENV, GITHUB_PATH or GITHUB_OUTPUT. The file variables are now quoted.

The scripts in ../scripts are outside the action directory and were not changed. Nothing was run to test these changes.

