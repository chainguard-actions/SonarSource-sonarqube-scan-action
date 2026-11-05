<!-- markdownlint-disable -->

# Hardening Report: SonarSource--sonarqube-scan-action--deprecated-c-cpp/v8.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **SonarSource--sonarqube-scan-action--deprecated-c-cpp/v8.4.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Setup action outputs' writes untrusted values to special environment files without the `printf '%s' ... | tr -d '\n\r'` sanitization. `echo "SONAR_HOST_URL=${SONAR_HOST_URL}" >> $GITHUB_ENV` forwards SONAR_HOST_URL, which is inherited from the calling workflow (case e). `echo "${SONAR_SCANNER_BIN_DIR}" >> $GITHUB_PATH` and `echo "${BUILD_WRAPPER_BIN_DIR}" >> $GITHUB_PATH` write paths built from steps.configure_paths.outputs.*, which come from inputs.installation-path. `echo "sonar-scanner-binary=${SONAR_SCANNER_BIN}" >> $GITHUB_OUTPUT` and `echo "build-wrapper-binary=${BUILD_WRAPPER_BIN}" >> $GITHUB_OUTPUT` write step-output-derived values. A newline in any of these values could inject extra env vars, PATH entries or outputs.

Locations:

- `action.yml:116`
- `action.yml:120`
- `action.yml:124`
- `action.yml:128`
- `action.yml:132`

### github-env-injection (severity: high)

Step 'Configure paths' sends the output of configure_paths.sh straight into $GITHUB_OUTPUT: `run: ${GITHUB_ACTION_PATH}/../scripts/configure_paths.sh >> $GITHUB_OUTPUT`. The script builds its values from INSTALL_PATH (inputs.installation-path) and from step outputs, and none of them pass through the printf|tr -d '\n\r' sanitization before the write.

Locations:

- `action.yml:69`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Setup action outputs: SONAR_HOST_URL and the four realpath-derived values (two PATH entries, two outputs) now go through printf '%s' | tr -d '\n\r' before they are written to GITHUB_ENV, GITHUB_PATH and GITHUB_OUTPUT. Each realpath result is captured on its own line first, so a realpath failure still stops the step under errexit. Configure paths: the script's output is captured (a script failure still fails the step) and read line by line. Only key=value lines for the 7 keys that action.yml reads are kept (sonar-scanner-dir/bin/url/sha, build-wrapper-dir/bin/url), and only the first line for each key. Each value is stripped of CR/LF before it is written. Two caveats: (1) scripts/configure_paths.sh is outside the action directory and I could not read it, so the key list comes from the outputs action.yml references. Any other key the script prints is now dropped. (2) The changes were not run or tested.

