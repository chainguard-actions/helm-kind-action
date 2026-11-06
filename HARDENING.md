<!-- markdownlint-disable -->

# Hardening Report: helm--kind-action/v1.15.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **helm--kind-action/v1.15.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. kind.sh writes user-controlled values to $GITHUB_PATH without sanitization. The variable `kind_dir` is derived from `${version}` (sourced from `INPUT_VERSION`, a user-supplied input) and `kubectl_dir` is derived from `${kubectl_version}` (sourced from `INPUT_KUBECTL_VERSION`). Both are written directly to `$GITHUB_PATH` via `echo "${kind_dir}" >> "${GITHUB_PATH}"` and `echo "${kubectl_dir}" >> "${GITHUB_PATH}"` with no preceding `printf '%s' ... | tr -d '\n\r'` sanitization step. An attacker can inject newlines into these inputs to poison the PATH or inject arbitrary environment variables.

Locations:

- `kind.sh:83`
- `kind.sh:89`

### github-env-injection (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. registry.sh writes user-controlled values to $GITHUB_OUTPUT without sanitization. The line `echo "LOCAL_REGISTRY=$registry_name:$registry_port" >> "$GITHUB_OUTPUT"` writes `$registry_name` (sourced from `INPUT_REGISTRY_NAME`) and `$registry_port` (sourced from `INPUT_REGISTRY_PORT`) — both user-supplied inputs — directly to $GITHUB_OUTPUT with no preceding `printf '%s' ... | tr -d '\n\r'` sanitization. An attacker can inject newlines to set arbitrary output variables or poison subsequent steps.

Locations:

- `registry.sh:120`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed kind.sh without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/kind.sh b/kind.sh
index eea0dea..40c2261 100755
--- a/kind.sh
+++ b/kind.sh
@@ -80,7 +80,8 @@ main() {
     fi
 
     echo 'Adding kind directory to PATH...'
-    echo "${kind_dir}" >> "${GITHUB_PATH}"
+    safe_kind_dir=$(printf '%s' "${kind_dir}" | tr -d '\n\r')
+    echo "${safe_kind_dir}" >> "${GITHUB_PATH}"
 
     local kubectl_dir="${cache_dir}/kubectl/bin/"
     if [[ ! -x "${kubectl_dir}/kubectl" ]]; then
@@ -88,7 +89,8 @@ main() {
     fi
 
     echo 'Adding kubectl directory to PATH...'
-    echo "${kubectl_dir}" >> "${GITHUB_PATH}"
+    safe_kubectl_dir=$(printf '%s' "${kubectl_dir}" | tr -d '\n\r')
+    echo "${safe_kubectl_dir}" >> "${GITHUB_PATH}"
 
     "${kind_dir}/kind" version
     "${kubectl_dir}/kubectl" version --client=true

```

Locations:

- `kind.sh`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed registry.sh without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/registry.sh b/registry.sh
index ff40b27..70412d0 100755
--- a/registry.sh
+++ b/registry.sh
@@ -131,7 +131,9 @@ create_registry() {
     echo "127.0.0.1 $registry_name" | sudo tee -a /etc/hosts
 
     # Write registry address to output for subsequent steps
-    echo "LOCAL_REGISTRY=$registry_name:$registry_port" >> "$GITHUB_OUTPUT"
+    safe_registry_name=$(printf '%s' "$registry_name" | tr -d '\n\r')
+    safe_registry_port=$(printf '%s' "$registry_port" | tr -d '\n\r')
+    echo "LOCAL_REGISTRY=${safe_registry_name}:${safe_registry_port}" >> "$GITHUB_OUTPUT"
 }
 
 connect_registry() {

```

Locations:

- `registry.sh`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, prior-fix

**Notes:**

kind.sh: kind_dir and kubectl_dir now go through printf '%s' | tr -d '\n\r' before being written to $GITHUB_PATH. registry.sh: registry_name and registry_port are cleaned the same way before LOCAL_REGISTRY is written to $GITHUB_OUTPUT. These are the same changes as the previous harden's patch.

