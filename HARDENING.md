<!-- markdownlint-disable -->

# Hardening Report: SonarSource--sonarqube-scan-action/v8.5.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **SonarSource--sonarqube-scan-action/v8.5.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

deprecated-c-cpp/action.yml, step 'Configure paths': the line `run: ${GITHUB_ACTION_PATH}/../scripts/configure_paths.sh >> $GITHUB_OUTPUT` sends the script's whole stdout to $GITHUB_OUTPUT. The script echoes values built from untrusted sources without `printf '%s' ... | tr -d '\n\r'`: INSTALL_PATH (from inputs.installation-path) in sonar-scanner-dir, sonar-scanner-bin, build-wrapper-dir and build-wrapper-bin; SONAR_HOST_URL (inherited from the calling workflow's environment) in build-wrapper-url; and the steps.*.outputs-derived SONAR_SCANNER_URL_*/SHA_* values. If any of these contains a newline, the caller can inject extra step outputs.

Locations:

- `deprecated-c-cpp/action.yml:69`
- `scripts/configure_paths.sh:82`

### github-env-injection (severity: high)

deprecated-c-cpp/action.yml, step 'Setup action outputs': `echo "SONAR_HOST_URL=${SONAR_HOST_URL}" >> $GITHUB_ENV` passes on SONAR_HOST_URL, which the composite action inherits from the calling workflow, without sanitizing it. In the same step, paths built from steps.configure_paths.outputs.* (which include inputs.installation-path) go to $GITHUB_PATH (`echo "${SONAR_SCANNER_BIN_DIR}" >> $GITHUB_PATH`, `echo "${BUILD_WRAPPER_BIN_DIR}" >> $GITHUB_PATH`) and to $GITHUB_OUTPUT (sonar-scanner-binary, build-wrapper-binary). None of these writes uses `printf '%s' ... | tr -d '\n\r'` first.

Locations:

- `deprecated-c-cpp/action.yml:116`
- `deprecated-c-cpp/action.yml:120`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Both findings are fixed. No tests were run.

- **`scripts/configure_paths.sh`:** I added a small helper, `emit_output`, that removes newlines with `printf '%s' | tr -d '\n\r'` before printing each `name=value` line. All seven outputs now go through it: sonar-scanner url, sha, dir and bin, and build-wrapper url, dir and bin. This covers values built from `INSTALL_PATH`, `SONAR_HOST_URL` and `SONAR_SCANNER_*`.
- **`deprecated-c-cpp/action.yml`, step 'Setup action outputs':** newlines are now removed from `SONAR_HOST_URL` before it is written to `GITHUB_ENV`. The same applies to the two bin directories written to `GITHUB_PATH` and the two binary paths written to `GITHUB_OUTPUT`. Each `realpath` call still runs as its own assignment, so if it fails the step still stops, as upstream's does. I also put quotes around `$GITHUB_ENV`, `$GITHUB_PATH` and `$GITHUB_OUTPUT`.

The `::error::` lines in `configure_paths.sh` still go to `GITHUB_OUTPUT`, as they do upstream. I left them alone because they contain only the runner's OS and architecture names.

