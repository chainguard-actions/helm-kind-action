<!-- markdownlint-disable -->

# Hardening Report: helm--kind-action/v1.14.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **helm--kind-action/v1.14.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

kind.sh writes `kind_dir` to `$GITHUB_PATH` without sanitization. `kind_dir` is derived from `${RUNNER_TOOL_CACHE}/kind/${version}/${arch}`, where `version` comes from the `--version` argument (sourced from `INPUT_VERSION`, a caller-controlled action input) and `RUNNER_TOOL_CACHE` is an inherited workflow-controlled environment variable. Neither value is passed through `printf '%s' ... | tr -d '\n\r'` before the write: `echo "${kind_dir}" >> "${GITHUB_PATH}"`. An attacker-controlled newline in `INPUT_VERSION` or `RUNNER_TOOL_CACHE` could inject arbitrary entries into `$GITHUB_PATH`.

Locations:

- `kind.sh:75`

### github-env-injection (severity: high)

kind.sh writes `kubectl_dir` to `$GITHUB_PATH` without sanitization. `kubectl_dir` is derived from `${RUNNER_TOOL_CACHE}/kind/${version}/${arch}`, where `version` comes from `INPUT_KUBECTL_VERSION` (a caller-controlled action input) and `RUNNER_TOOL_CACHE` is an inherited workflow-controlled environment variable. The write `echo "${kubectl_dir}" >> "${GITHUB_PATH}"` is not preceded by the required `printf '%s' ... | tr -d '\n\r'` sanitization step.

Locations:

- `kind.sh:81`

### github-env-injection (severity: high)

registry.sh writes `LOCAL_REGISTRY=$registry_name:$registry_port` to `$GITHUB_OUTPUT` without sanitization. Both `registry_name` and `registry_port` are sourced from caller-controlled action inputs (`INPUT_REGISTRY_NAME` and `INPUT_REGISTRY_PORT` respectively, passed via command-line arguments). The write `echo "LOCAL_REGISTRY=$registry_name:$registry_port" >> "$GITHUB_OUTPUT"` is not preceded by the required `printf '%s' ... | tr -d '\n\r'` sanitization step, allowing a newline in either input to inject arbitrary key=value pairs into `$GITHUB_OUTPUT`.

Locations:

- `registry.sh:116`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed three github-env-injection findings:
1. kind.sh: Sanitized `kind_dir` before writing to $GITHUB_PATH using `printf '%s' "${kind_dir}" | tr -d '\n\r' >> "${GITHUB_PATH}"` followed by `echo >> "${GITHUB_PATH}"` to preserve the required newline terminator.
2. kind.sh: Same sanitization applied to `kubectl_dir` before writing to $GITHUB_PATH.
3. registry.sh: Sanitized both `registry_name` and `registry_port` into `safe_registry_name` and `safe_registry_port` using `printf '%s' ... | tr -d '\n\r'` before writing `LOCAL_REGISTRY=...` to $GITHUB_OUTPUT.

