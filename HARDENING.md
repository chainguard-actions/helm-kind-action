<!-- markdownlint-disable -->

# Hardening Report: helm--kind-action/v1.14.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **helm--kind-action/v1.14.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In kind.sh, the variables `kind_dir` and `kubectl_dir` are constructed from user-controlled inputs (`INPUT_VERSION` → `version`, `INPUT_KUBECTL_VERSION` → `kubectl_version`) as path components (e.g. `${RUNNER_TOOL_CACHE}/kind/${version}/${arch}`). These paths are written directly to `$GITHUB_PATH` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker supplying a newline-containing version string could inject arbitrary entries into the runner's PATH for subsequent steps.

Locations:

- `kind.sh:80`
- `kind.sh:86`

### github-env-injection (severity: high)

In registry.sh, the `create_registry` function writes `LOCAL_REGISTRY=$registry_name:$registry_port` to `$GITHUB_OUTPUT`. Both `registry_name` and `registry_port` are derived from user-controlled inputs (`INPUT_REGISTRY_NAME` and `INPUT_REGISTRY_PORT` via command-line arguments from main.sh). No sanitization (`printf '%s' ... | tr -d '\n\r'`) is applied before the write, allowing an attacker to inject arbitrary key=value pairs into the GitHub output context via a newline in the input.

Locations:

- `registry.sh:109`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings:
1. kind.sh (lines 80, 86): Replaced bare `echo "${kind_dir}" >> "${GITHUB_PATH}"` and `echo "${kubectl_dir}" >> "${GITHUB_PATH}"` with `printf '%s' "${...}" | tr -d '\n\r' >> "${GITHUB_PATH}"` followed by `echo >> "${GITHUB_PATH}"` to strip newlines from user-controlled version-derived paths before writing to GITHUB_PATH.
2. registry.sh (line 109): Introduced `safe_registry_name` and `safe_registry_port` variables sanitized via `printf '%s' ... | tr -d '\n\r'`, then used those safe values when writing `LOCAL_REGISTRY=...` to GITHUB_OUTPUT, preventing newline injection from user-controlled INPUT_REGISTRY_NAME and INPUT_REGISTRY_PORT.

