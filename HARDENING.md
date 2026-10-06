<!-- markdownlint-disable -->

# Hardening Report: helm--kind-action/v1.15.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **helm--kind-action/v1.15.1** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In kind.sh, the `version` variable is populated from the `INPUT_VERSION` action input (user-controlled), which flows into `cache_dir` and then into `kind_dir` ("${RUNNER_TOOL_CACHE}/kind/${version}/${arch}/kind/bin/") and `kubectl_dir`. Both are written directly to `$GITHUB_PATH` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A caller can inject newlines into the version input to write arbitrary entries into GITHUB_PATH, hijacking the PATH for subsequent steps. Similarly, `kubectl_version` (from `INPUT_KUBECTL_VERSION`) flows into `kubectl_dir` → `$GITHUB_PATH` unsanitized.

Locations:

- `kind.sh:83`
- `kind.sh:89`

### github-env-injection (severity: high)

In registry.sh, the `registry_name` variable (sourced from the `INPUT_REGISTRY_NAME` action input, user-controlled) and `registry_port` (from `INPUT_REGISTRY_PORT`, user-controlled) are written directly to `$GITHUB_OUTPUT` as `echo "LOCAL_REGISTRY=$registry_name:$registry_port" >> "$GITHUB_OUTPUT"` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A caller supplying a newline-containing registry name or port can inject arbitrary key=value pairs into GITHUB_OUTPUT, poisoning outputs consumed by downstream steps.

Locations:

- `registry.sh:130`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings:
1. kind.sh (lines 83, 89): Added sanitization of `kind_dir` and `kubectl_dir` (derived from user-controlled `version`/`kubectl_version` inputs) before writing to $GITHUB_PATH. Used `safe_kind_dir=$(printf '%s' "${kind_dir}" | tr -d '\n\r')` and `safe_kubectl_dir=$(printf '%s' "${kubectl_dir}" | tr -d '\n\r')` patterns.
2. registry.sh (line 130): Added sanitization of `registry_name` and `registry_port` (from user-controlled inputs) before writing to $GITHUB_OUTPUT. Used `safe_registry_name=$(printf '%s' "$registry_name" | tr -d '\n\r')` and `safe_registry_port=$(printf '%s' "$registry_port" | tr -d '\n\r')` patterns, then wrote the sanitized values.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted variable expansion in registry.sh at line 117. Changed `$registry_image` to `"$registry_image"` in the `docker run` command within the `create_registry()` function. This prevents word-splitting and glob expansion on the user-controlled `INPUT_REGISTRY_IMAGE` action input value, eliminating the potential for shell metacharacter injection.

