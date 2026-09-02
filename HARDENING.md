<!-- markdownlint-disable -->

# Hardening Report: helm--kind-action/v1.15.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **helm--kind-action/v1.15.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In kind.sh, the `kind_dir` variable (which embeds `$version` sourced from `INPUT_VERSION`, a user-controlled action input) is written directly to `$GITHUB_PATH` without sanitization (`printf '%s' ... | tr -d '\n\r'`). Similarly, `kubectl_dir` (which embeds `$kubectl_version` from `INPUT_KUBECTL_VERSION`) is also written to `$GITHUB_PATH` unsanitized. A newline in either input value could inject arbitrary entries into the runner's PATH.

Locations:

- `kind.sh:82`
- `kind.sh:91`

### github-env-injection (severity: high)

In registry.sh, the `create_registry` function writes `LOCAL_REGISTRY=$registry_name:$registry_port` to `$GITHUB_OUTPUT` without sanitization. Both `registry_name` and `registry_port` are sourced from user-controlled action inputs (`INPUT_REGISTRY_NAME` and `INPUT_REGISTRY_PORT`). A newline embedded in either value could inject additional key=value pairs into GITHUB_OUTPUT, allowing an attacker to set arbitrary outputs.

Locations:

- `registry.sh:129`

### script-injection (severity: high)

Rule (b) violation: In the `test-with-registry` and `test-with-registry-and-delete-enabled` jobs, the env var `$LOCAL_REGISTRY` (set from `${{ steps.kind.outputs.LOCAL_REGISTRY }}`, a workflow-controllable step output) is used unquoted in shell commands such as `docker tag busybox $LOCAL_REGISTRY/localbusybox`, `docker push $LOCAL_REGISTRY/localbusybox`, and `kubectl create job test --image=$LOCAL_REGISTRY/localbusybox`. Unquoted expansion allows shell metacharacter injection if the output value contains spaces, semicolons, or other special characters.

Locations:

- `.github/workflows/test.yaml:228`
- `.github/workflows/test.yaml:229`
- `.github/workflows/test.yaml:232`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed three security findings: (1) kind.sh: sanitized kind_dir and kubectl_dir before writing to GITHUB_PATH using `printf '%s' ... | tr -d '\n\r'` to strip embedded newlines that could inject arbitrary PATH entries; (2) registry.sh: sanitized registry_name and registry_port before writing LOCAL_REGISTRY to GITHUB_OUTPUT using the same tr-based stripping approach; (3) test.yaml: added double-quotes around all $LOCAL_REGISTRY shell variable expansions in both test-with-registry and test-with-registry-and-delete-enabled jobs to prevent shell metacharacter injection (the variable was already correctly placed in the env: block).

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted `$registry_image` variable in `hardened/action/registry.sh` at line 113. Changed `$registry_image` to `"$registry_image"` in the `docker run` command. This prevents word-splitting and glob expansion on the caller-controlled `registry_image` input, eliminating the potential for command injection via shell metacharacters.

