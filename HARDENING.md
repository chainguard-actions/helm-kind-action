<!-- markdownlint-disable -->

# Hardening Report: helm--kind-action/v1.14.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **helm--kind-action/v1.14.0** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

kind.sh writes unsanitized user-controlled input to $GITHUB_PATH. The variable `kind_dir` is constructed from `${version}` (sourced from INPUT_VERSION, a caller-controlled action input) and written directly to $GITHUB_PATH without the required `printf '%s' ... | tr -d '\n\r'` sanitization step. An attacker can inject newlines to add arbitrary entries to PATH. The same issue applies to `kubectl_dir` on the following write.

Locations:

- `kind.sh:83`
- `kind.sh:91`

### github-env-injection (severity: high)

registry.sh writes unsanitized user-controlled inputs to $GITHUB_OUTPUT. The value `LOCAL_REGISTRY=$registry_name:$registry_port` is written directly to $GITHUB_OUTPUT, where `registry_name` comes from INPUT_REGISTRY_NAME and `registry_port` from INPUT_REGISTRY_PORT — both caller-controlled action inputs. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write, allowing newline injection to poison subsequent step outputs.

Locations:

- `registry.sh:133`

### permissions (severity: medium)

The workflow file .github/workflows/test.yaml has no top-level `permissions:` key and none of its 14 jobs define a job-level `permissions:` block. This means the workflow runs with the default (potentially broad) GITHUB_TOKEN permissions. A minimal explicit permissions block (e.g., `permissions: {}` or specific scopes) should be added.

Locations:

- `.github/workflows/test.yaml:1`

### script-injection (severity: high)

Rule (b) violation: The env var LOCAL_REGISTRY is populated from `${{ steps.kind.outputs.LOCAL_REGISTRY }}` (a steps.*.outputs.* context, which is workflow-controllable) and then expanded unquoted inside run: shell commands in two jobs. Unquoted shell expansion allows an attacker-controlled value to inject shell metacharacters. Affected lines include: `docker tag busybox $LOCAL_REGISTRY/localbusybox`, `docker push $LOCAL_REGISTRY/localbusybox`, `kubectl create job test --image=$LOCAL_REGISTRY/localbusybox` (test-with-registry job), and `docker tag busybox $LOCAL_REGISTRY/localbusybox`, `DIGEST=$(docker push $LOCAL_REGISTRY/localbusybox | ...)`, `curl -X DELETE $LOCAL_REGISTRY/v2/...` (test-with-registry-and-delete-enabled job). All occurrences of $LOCAL_REGISTRY should be double-quoted as "$LOCAL_REGISTRY".

Locations:

- `.github/workflows/test.yaml:221`
- `.github/workflows/test.yaml:222`
- `.github/workflows/test.yaml:224`
- `.github/workflows/test.yaml:255`
- `.github/workflows/test.yaml:257`
- `.github/workflows/test.yaml:259`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, permissions, script-injection

**Notes:**

Fixed 4 findings across 3 files: (1) kind.sh lines 83 & 91: sanitized kind_dir and kubectl_dir with `printf '%s' ... | tr -d '\n\r'` before writing to GITHUB_PATH; (2) registry.sh line 133: sanitized registry_name and registry_port with `printf '%s' ... | tr -d '\n\r'` before writing LOCAL_REGISTRY to GITHUB_OUTPUT; (3) .github/workflows/test.yaml: added `permissions: {}` top-level block; (4) .github/workflows/test.yaml: double-quoted all 6 occurrences of $LOCAL_REGISTRY in run: commands across the test-with-registry and test-with-registry-and-delete-enabled jobs.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell variable expansion in registry.sh. Changed `$registry_image` to `"$registry_image"` in the `docker run` command inside the `create_registry()` function (line 107). This prevents shell word splitting and command injection via the `INPUT_REGISTRY_IMAGE` workflow input, which is passed through `main.sh` as `--registry-image` to `registry.sh` and stored in the `registry_image` local variable.

