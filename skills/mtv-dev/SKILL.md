---
name: mtv-dev
description: Walk the user through the manual Forklift/MTV feature test flow against a live OpenShift cluster — test-<desc> namespace, host provider, vSphere provider from GOVC_* env vars, migration plan — one approved command at a time. Use when the user wants to test a feature they just developed, run the manual test flow, create test providers, or stand up a fresh test namespace.
---

# mtv-dev — Manual feature test flow

Order: optional controller/catalog refresh → `test-<description>` namespace → host provider → vSphere (or other sources in [ref-providers.md](ref-providers.md)) → migration plan for the feature.

**Env:** `GOVC_URL`, `GOVC_USERNAME`, `GOVC_PASSWORD` for vSphere; `openssl` for CA fetch.

## Operating rules

- **Approve mutating commands** before running; read-only is fine without asking.
- **Forklift resources:** `oc mtv` for create/delete/get/describe/watch — not `oc get/describe` on `*.forklift.konveyor.io`. Plain `oc` for namespaces, `forklift-controller` deploy/pods, operator catalog.
- **Automation:** `oc wait … --for=condition=Ready` after `oc mtv create` ([below](#automation-wait-for-ready)).
- **Reuse** an existing `test-*` namespace that already has providers when possible.
- Names: `test-<description>`, providers `host` / `vsphere`, plan `<description>-plan` unless the user overrides.

## Automation — wait for Ready

`oc mtv` has no `wait`. Scripts use the CR API:

```bash
NS=test-<description>
oc wait provider.forklift.konveyor.io/<name> -n "$NS" --for=condition=Ready --timeout=300s
oc wait plan.forklift.konveyor.io/<plan-name> -n "$NS" --for=condition=Ready --timeout=300s
```

On failure: `oc mtv describe provider|plan --name … -n "$NS"`. Bump `--timeout` for large vSphere inventory.

## Preflight

```bash
oc get ns -o name | grep '^namespace/test-' || true
oc mtv get provider -n <namespace> || true
```

## Step 0a — Controller image refresh (optional)

Re-pushed **same tag:** delete controller pod so the node pulls again.

```bash
oc get deploy forklift-controller -n konveyor-forklift -o jsonpath='{.spec.template.spec.containers[*].image}'
oc delete pod -n konveyor-forklift -l control-plane=controller-manager
```

New **tag:** `oc set image deployment/forklift-controller -n konveyor-forklift …` then delete the pod.

## Step 0b — Operator bundle + index (optional, API/CRD changes)

From forklift repo root — **bundle before index** (index embeds the bundle):

```bash
make push-operator-bundle-image REGISTRY=<registry> REGISTRY_ORG=<org> REGISTRY_TAG=<tag>
make push-operator-index-image  REGISTRY=<registry> REGISTRY_ORG=<org> REGISTRY_TAG=<tag>
make deploy-operator-index REGISTRY=<registry> REGISTRY_ORG=<org> REGISTRY_TAG=<tag>
```

`PLATFORM=linux/amd64` for single-arch builds. Then Step 0a if needed.

## Steps 1–5

```bash
oc create namespace test-<description>

oc mtv create provider --name host --type openshift -n test-<description>

# vSphere URL must be …/sdk; lab TLS skip: --provider-insecure-skip-tls instead of --cacert
oc mtv create provider --name vsphere --type vsphere \
  --url "https://${GOVC_URL}/sdk" \
  --username "${GOVC_USERNAME}" --password "${GOVC_PASSWORD}" \
  --cacert "$(fetch_ca_cert "${GOVC_URL}")" \
  -n test-<description>

oc mtv create plan --name <description>-plan --source vsphere --vms <vm> -n test-<description>
```

**vSphere CA** (host/port from `GOVC_URL`):

```bash
fetch_ca_cert() {
  local hostport host
  hostport=$(echo "$1" | sed -E 's|https?://||; s|/.*||')
  host="${hostport%%:*}"
  if ! echo "${hostport}" | grep -q ':'; then hostport="${hostport}:443"; fi
  openssl s_client -showcerts -servername "${host}" -connect "${hostport}" </dev/null 2>/dev/null \
    | openssl x509 -outform PEM
}
```

Other provider types and env vars: [ref-providers.md](ref-providers.md).

**Readiness (interactive):** `oc mtv get|describe provider|plan …` and `-w` on the resource. **Automation:** `oc wait` lines above (`host`, `vsphere`, `<description>-plan`).

## Cleanup (on request, user confirms)

Delete **plans before providers**. Deleting a plan does **not** remove migrated KubeVirt VMs — list with `oc mtv get plan --name … --vms` or `--vms-table`.

| Case | Action |
|------|--------|
| Failed / partial migration | `oc mtv delete plan --name … --clean-all -n …` |
| VMs to remove | `oc delete vm <name>` or `oc delete vm --all -n …`; PVCs may need `oc delete pvc` |
| Forklift wipe | `oc mtv delete plan --all --clean-all -n …` then `oc mtv delete provider --all -n …` |
| Teardown | `oc delete namespace test-<description>` (only after MTV objects gone; target VMs may live outside this NS) |

Named cleanup: plan (`--clean-all` if needed) → `vsphere` → `host` → namespace.
