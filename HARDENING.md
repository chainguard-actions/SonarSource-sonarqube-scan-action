<!-- markdownlint-disable -->

# Hardening Report: SonarSource--sonarqube-scan-action/v8.5.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **SonarSource--sonarqube-scan-action/v8.5.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

deprecated-c-cpp/action.yml, step 'Configure paths': `run: ${GITHUB_ACTION_PATH}/../scripts/configure_paths.sh >> $GITHUB_OUTPUT` sends the script's output straight to GITHUB_OUTPUT. That output contains values built from untrusted data: INSTALL_PATH (from inputs.installation-path), step outputs, and SONAR_HOST_URL, which the calling workflow sets and the composite action inherits. These include sonar-scanner-dir, sonar-scanner-bin, build-wrapper-url, build-wrapper-dir and build-wrapper-bin. None of them go through `printf '%s' ... | tr -d '\n\r'` first, so a newline in a value can inject extra outputs.

Locations:

- `deprecated-c-cpp/action.yml:69`
- `scripts/configure_paths.sh:82`

### github-env-injection (severity: high)

deprecated-c-cpp/action.yml, step 'Setup action outputs': `echo "SONAR_HOST_URL=${SONAR_HOST_URL}" >> $GITHUB_ENV` passes an env var inherited from the calling workflow into GITHUB_ENV without sanitising it. The same step also writes SONAR_SCANNER_BIN_DIR and BUILD_WRAPPER_BIN_DIR to $GITHUB_PATH, and sonar-scanner-binary and build-wrapper-binary to $GITHUB_OUTPUT. Those values come from step outputs that contain inputs.installation-path, and none of these writes uses `printf '%s' ... | tr -d '\n\r'` first.

Locations:

- `deprecated-c-cpp/action.yml:116`
- `deprecated-c-cpp/action.yml:120`
- `deprecated-c-cpp/action.yml:124`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

I fixed both github-env-injection findings. Nothing has been run or tested.

1. scripts/configure_paths.sh (the 'Configure paths' step): every value the script writes to GITHUB_OUTPUT now goes through a small helper that strips CR and LF characters from it. That covers sonar-scanner-url, sonar-scanner-sha, sonar-scanner-dir, sonar-scanner-bin, build-wrapper-url, build-wrapper-dir and build-wrapper-bin. The script's error messages and exit codes are the same as before. Only deprecated-c-cpp/action.yml calls this script.

2. deprecated-c-cpp/action.yml, 'Setup action outputs' step: SONAR_HOST_URL now has newlines stripped before it is written to GITHUB_ENV, and the same applies to the two directories added to GITHUB_PATH and the two binary paths written to GITHUB_OUTPUT. The 'realpath' calls stay on their own lines and the stripping happens on a separate line afterwards, so a failing realpath still stops the step as it did upstream.

I also put quotes around $GITHUB_OUTPUT, $GITHUB_ENV and $GITHUB_PATH wherever these two steps write to them.

