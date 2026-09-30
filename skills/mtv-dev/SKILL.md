---
name: mtv-dev
description: Walk the user through the manual Forklift/MTV feature test flow against a live OpenShift cluster — test-<desc> namespace, host provider, vSphere provider from GOVC_* env vars, migration plan — one approved command at a time. Use when the user wants to test a feature they just developed, run the manual test flow, create test providers, or stand up a fresh test namespace.
---

# mtv-dev — Manual Feature Test Flow

Runs the manual end-to-end test flow for the MTV feature under development, against a live cluster:

1. (optional) refresh the controller, or rebuild + push the operator bundle/index (when the API changed)
2. Namespace `test-<description>`
3. Host provider (local OpenShift, auto-detected)
4. vSphere provider (credentials from `GOVC_*` env vars)
5. Additional source providers as needed (see [ref-providers.md](ref-providers.md))
6. A migration plan that exercises the feature under development

## Operating Rules

- **Suggest, then run.** For every mutating command, show the exact command and wait for the user to approve it (per step, or for the whole run). Read-only commands may run freely.
- **Forklift CRs via `oc mtv`.** For providers, plans, migrations, and other Forklift resources, use `oc mtv` for create/delete/get/describe and interactive readiness (`-w` / `--watch`). Do not use vanilla `oc get` / `oc describe` on `*.forklift.konveyor.io`. Use plain `oc` for cluster objects (namespaces, deployments, pods, operator catalog, etc.). For **automation** (scripts, CI, unattended runs), block on `Ready` with `oc wait` on the CR — see [Automation — wait for Ready](#automation--wait-for-ready).
- **Reuse before create.** Before creating a namespace, check for an existing `test-*` namespace that already has the required providers — if one exists, confirm with the user and reuse it, skipping the steps it already satisfies.
- **Gate on readiness.** After each step, stop and let the user verify readiness before moving on. If they ask how to check, show that step's readiness commands.
- Derive namespace/provider/plan names from the feature's `<description>` unless the user specifies otherwise.

## Required CLI

`oc` with the `mtv` plugin, plus `openssl` (to fetch the vSphere CA cert):

```bash
oc mtv --help
```

Verify the vSphere credentials are exported: `GOVC_URL`, `GOVC_USERNAME`, `GOVC_PASSWORD`.

## Automation — wait for Ready

`oc mtv` has no `wait` subcommand. After `oc mtv create`, scripts and CI should gate on the Forklift CR `Ready` condition with `oc wait` (same API resource the controller updates):

```bash
NS=test-<description>

# Provider (resource name = --name from oc mtv create provider)
oc wait provider.forklift.konveyor.io/<provider-name> -n "$NS" \
  --for=condition=Ready --timeout=300s

# Plan (resource name = --name from oc mtv create plan)
oc wait plan.forklift.konveyor.io/<plan-name> -n "$NS" \
  --for=condition=Ready --timeout=300s
```

Examples for this flow:

```bash
oc wait provider.forklift.konveyor.io/host -n "$NS" --for=condition=Ready --timeout=300s
oc wait provider.forklift.konveyor.io/vsphere -n "$NS" --for=condition=Ready --timeout=300s
oc wait plan.forklift.konveyor.io/<description>-plan -n "$NS" --for=condition=Ready --timeout=300s
```

- `--for=condition=Ready` succeeds when `status.conditions` includes `type=Ready` and `status=True`.
- Increase `--timeout` for slow inventory (large vSphere) or cold clusters.
- On failure, inspect with `oc mtv describe provider --name <name> -n "$NS"` (or `plan`) — do not rely on `oc describe` of the raw CR in routine flows.
- Other gates (e.g. migration `Running` / `Succeeded`) use the same pattern with the appropriate `*.forklift.konveyor.io/<name>` kind and condition.

## Preflight

```bash
# Existing test namespaces?
oc get ns -o name | grep '^namespace/test-' || true

# Providers already in a candidate namespace?
oc mtv get provider -n <namespace> || true
```

## Step 0a — Refresh a newly pushed controller image (optional)

When the user pushed a new controller image and the cluster runs the old build:

```bash
# Verify the deployment points at the new build
oc get deploy forklift-controller -n konveyor-forklift \
  -o jsonpath='{.spec.template.spec.containers[*].image}'

# If the tag is unchanged but the image was re-pushed, delete the pod to pick up the new build
oc delete pod -n konveyor-forklift -l control-plane=controller-manager

oc get pods -n konveyor-forklift -w
```

If the image *tag* changed, update the deployment first (`oc set image deployment/forklift-controller -n konveyor-forklift <container>=<image>`), then delete the pod.

## Step 0b — Rebuild + push operator bundle and index (optional)

Only when the feature changed the Forklift API/CRD. Run from the forklift repo root; **order matters** — bundle first, index second (the index embeds the bundle image):

```bash
make push-operator-bundle-image REGISTRY=<registry> REGISTRY_ORG=<org> REGISTRY_TAG=<tag>
make push-operator-index-image  REGISTRY=<registry> REGISTRY_ORG=<org> REGISTRY_TAG=<tag>
```

Then point the cluster's operator catalog at the new index:

```bash
make deploy-operator-index REGISTRY=<registry> REGISTRY_ORG=<org> REGISTRY_TAG=<tag>
```

Add `PLATFORM=linux/amd64` for a single-arch build. After the catalog updates, refresh the controller per Step 0a if needed.

## Step 1 — Namespace

```bash
oc create namespace test-<description>
```

Readiness: `oc get ns test-<description>`

## Step 2 — Host provider

The local OpenShift provider; it auto-detects the current cluster (no URL/token needed). It is normally the migration *target*.

```bash
oc mtv create provider --name host --type openshift -n test-<description>
```

Readiness:

```bash
oc mtv get provider -n test-<description>
oc mtv describe provider --name host -n test-<description>
oc mtv get provider --name host -n test-<description> -w
```

Automation: `oc wait provider.forklift.konveyor.io/host -n test-<description> --for=condition=Ready --timeout=300s`

## Step 3 — vSphere provider (GOVC_*)

Fetch the CA cert from vCenter first:

```bash
fetch_ca_cert() {
  local hostport host
  hostport=$(echo "$1" | sed -E 's|https?://||; s|/.*||')
  host="${hostport%%:*}"
  if ! echo "${hostport}" | grep -q ':'; then
    hostport="${hostport}:443"
  fi
  openssl s_client -showcerts -servername "${host}" \
    -connect "${hostport}" </dev/null 2>/dev/null \
    | openssl x509 -outform PEM
}
```

Then create the provider:

```bash
oc mtv create provider --name vsphere --type vsphere \
  --url "https://${GOVC_URL}/sdk" \
  --username "${GOVC_USERNAME}" \
  --password "${GOVC_PASSWORD}" \
  --cacert "$(fetch_ca_cert "${GOVC_URL}")" \
  -n test-<description>
```

For labs with self-signed certs, replace `--cacert ...` with `--provider-insecure-skip-tls`.

Readiness:

```bash
oc mtv get provider -n test-<description>
oc mtv describe provider --name vsphere -n test-<description>
oc mtv get provider --name vsphere -n test-<description> -w
```

Automation: `oc wait provider.forklift.konveyor.io/vsphere -n test-<description> --for=condition=Ready --timeout=300s`

## Step 4 — Other source providers (as needed)

Full per-type commands in [ref-providers.md](ref-providers.md).

| Type | Required env vars |
|------|-------------------|
| ovirt / RHV | `RHV_URL`, `RHV_USERNAME`, `RHV_PASSWORD` |
| openstack | `OSP_URL`, `OSP_USERNAME`, `OSP_PASSWORD`, `OSP_DOMAIN_NAME`, `OSP_PROJECT_NAME`, `OSP_REGION_NAME` |
| ova | `OVA_URL` |
| openshift (remote source) | `SOURCE_OCP_URL`, `SOURCE_OCP_TOKEN` |
| hyperv | `HV_URL`, `HV_USERNAME`, `HV_PASSWORD`, `HV_SMB_URL` |
| ec2 | `EC2_REGION`, `EC2_ACCESS_KEY_ID`, `EC2_SECRET_ACCESS_KEY`, `EC2_TARGET_AZ`, `EC2_TARGET_REGION` |

Create each with its `oc mtv create provider --type <t>` command; wait for `Ready` interactively (`oc mtv get provider --name <name> -w`) or in automation (`oc wait provider.forklift.konveyor.io/<name> -n test-<description> --for=condition=Ready --timeout=300s`).

## Step 5 — Migration plan

Assemble the plan that exercises the feature under development. Base form — adjust source provider, VM selection, and any feature-specific flags (ask the user which VMs and options the feature needs):

```bash
oc mtv create plan --name <description>-plan --source vsphere --vms <vm> -n test-<description>
```

Readiness:

```bash
oc mtv get plan -n test-<description>
oc mtv describe plan --name <description>-plan -n test-<description>
oc mtv get plan --name <description>-plan -n test-<description> -w
```

Automation: `oc wait plan.forklift.konveyor.io/<description>-plan -n test-<description> --for=condition=Ready --timeout=300s`

## Cleanup (on request)

Confirm each destructive step with the user before running it. Delete **plans before providers** (plans reference providers).

### Target VMs the plan created

Deleting a plan does **not** remove KubeVirt VMs that already migrated to the target. List them first (target namespace is usually `test-<description>` — confirm on the plan):

```bash
oc mtv get plan --name <description>-plan --vms -n test-<description>
oc mtv describe plan --name <description>-plan --with-vms -n test-<description>
oc mtv get plan --vms-table -n test-<description>
```

Ask whether to keep or remove those VMs.

- **Failed / partial migration** — delete the plan with `--clean-all` so MTV archives the plan, removes target VMs from the failed migration, then deletes the plan:

```bash
oc mtv delete plan --name <description>-plan --clean-all -n test-<description>
```

- **Succeeded migration (or any VM still on the cluster)** — delete KubeVirt VMs in the target namespace before or after dropping the plan:

```bash
oc delete vm <vm-name> -n test-<description>
```

Or wipe every VM in the test namespace when the whole run is disposable:

```bash
oc delete vm --all -n test-<description>
```

PVCs/DataVolumes for those VMs may remain; list with `oc get pvc -n test-<description>` and delete if disk cleanup is required.

### Forklift resources and namespace

By name (add `--clean-all` on `delete plan` when failed-migration VMs should go with the plan):

```bash
oc mtv delete plan --name <description>-plan --clean-all -n test-<description>
oc mtv delete provider --name vsphere -n test-<description>
oc mtv delete provider --name host -n test-<description>
oc delete namespace test-<description>
```

Or remove all plans and providers in the test namespace (`--all`; use `--clean-all` on `delete plan` when cleaning failed-migration VMs):

```bash
oc mtv delete plan --all --clean-all -n test-<description>
oc mtv delete provider --all -n test-<description>
oc delete namespace test-<description>
```

`oc delete namespace` removes namespaced objects in `test-<description>` (including VMs and PVCs) but only after plans/providers are gone and nothing blocks termination — prefer explicit VM deletion when the target namespace is not the test namespace.
