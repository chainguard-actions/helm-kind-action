<!-- markdownlint-disable -->

# Hardening Report: helm--kind-action/v1.15.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **helm--kind-action/v1.15.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. In kind.sh, the `version` variable is populated from the `INPUT_VERSION` action input (user-controlled), which flows into `cache_dir` and then into `kind_dir` ("${RUNNER_TOOL_CACHE}/kind/${version}/${arch}/kind/bin/") and `kubectl_dir`. Both are written directly to `$GITHUB_PATH` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A caller can inject newlines into the version input to write arbitrary entries into GITHUB_PATH, hijacking the PATH for subsequent steps. Similarly, `kubectl_version` (from `INPUT_KUBECTL_VERSION`) flows into `kubectl_dir` → `$GITHUB_PATH` unsanitized.

Locations:

- `kind.sh:83`
- `kind.sh:89`

### github-env-injection (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. In registry.sh, the `registry_name` variable (sourced from the `INPUT_REGISTRY_NAME` action input, user-controlled) and `registry_port` (from `INPUT_REGISTRY_PORT`, user-controlled) are written directly to `$GITHUB_OUTPUT` as `echo "LOCAL_REGISTRY=$registry_name:$registry_port" >> "$GITHUB_OUTPUT"` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A caller supplying a newline-containing registry name or port can inject arbitrary key=value pairs into GITHUB_OUTPUT, poisoning outputs consumed by downstream steps.

Locations:

- `registry.sh:130`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed kind.sh without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/kind.sh b/kind.sh
index 2ea8ff1..dc4dbc5 100755
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
index ff40b27..e0a85a3 100755
--- a/registry.sh
+++ b/registry.sh
@@ -125,13 +125,15 @@ create_registry() {
         -p "${registry_port}:${registry_port}" \
         -e REGISTRY_HTTP_ADDR="0.0.0.0:${registry_port}" \
         -e REGISTRY_STORAGE_DELETE_ENABLED="$enable_delete" \
-        $registry_image
+        "$registry_image"
 
     # Local registry is available at $registry_name:$registry_port
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

kind.sh: strip CR/LF from kind_dir and kubectl_dir before appending to GITHUB_PATH. registry.sh: strip CR/LF from registry_name/port before writing LOCAL_REGISTRY to GITHUB_OUTPUT; quoted $registry_image in docker run (as in the prior harden).

