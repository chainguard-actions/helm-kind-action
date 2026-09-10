<!-- markdownlint-disable -->

# Hardening Report: helm--kind-action/v1.15.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **helm--kind-action/v1.15.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

kind.sh writes user-controlled values to $GITHUB_PATH without sanitization. The variable `kind_dir` is derived from `${version}` (sourced from `INPUT_VERSION`, a user-supplied input) and `kubectl_dir` is derived from `${kubectl_version}` (sourced from `INPUT_KUBECTL_VERSION`). Both are written directly to `$GITHUB_PATH` via `echo "${kind_dir}" >> "${GITHUB_PATH}"` and `echo "${kubectl_dir}" >> "${GITHUB_PATH}"` with no preceding `printf '%s' ... | tr -d '\n\r'` sanitization step. An attacker can inject newlines into these inputs to poison the PATH or inject arbitrary environment variables.

Locations:

- `kind.sh:83`
- `kind.sh:89`

### github-env-injection (severity: high)

registry.sh writes user-controlled values to $GITHUB_OUTPUT without sanitization. The line `echo "LOCAL_REGISTRY=$registry_name:$registry_port" >> "$GITHUB_OUTPUT"` writes `$registry_name` (sourced from `INPUT_REGISTRY_NAME`) and `$registry_port` (sourced from `INPUT_REGISTRY_PORT`) — both user-supplied inputs — directly to $GITHUB_OUTPUT with no preceding `printf '%s' ... | tr -d '\n\r'` sanitization. An attacker can inject newlines to set arbitrary output variables or poison subsequent steps.

Locations:

- `registry.sh:120`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings:
1. kind.sh (lines 83, 89): Sanitized `kind_dir` and `kubectl_dir` before writing to $GITHUB_PATH using `safe_kind_dir=$(printf '%s' "${kind_dir}" | tr -d '\n\r')` and `safe_kubectl_dir=$(printf '%s' "${kubectl_dir}" | tr -d '\n\r')` respectively, then writing the safe variables instead.
2. registry.sh (line 120): Sanitized `registry_name` and `registry_port` before writing to $GITHUB_OUTPUT using `safe_registry_name=$(printf '%s' "$registry_name" | tr -d '\n\r')` and `safe_registry_port=$(printf '%s' "$registry_port" | tr -d '\n\r')`, then composing the output value from the sanitized variables.

