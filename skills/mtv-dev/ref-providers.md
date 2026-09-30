# Provider Creation Commands

Use these `oc mtv create provider` commands in the test flow. Pick the block that
matches the source provider type. All commands take `-n <namespace>` — the
`test-<description>` namespace from the flow.

## vSphere (with CA cert — default)

```bash
oc mtv create provider --name vsphere --type vsphere \
  --url "https://${GOVC_URL}/sdk" \
  --username "${GOVC_USERNAME}" \
  --password "${GOVC_PASSWORD}" \
  --cacert "$(fetch_ca_cert "${GOVC_URL}")" \
  -n "${NS}"
```

## vSphere (insecure — lab only)

```bash
oc mtv create provider --name vsphere --type vsphere \
  --url "https://${GOVC_URL}/sdk" \
  --username "${GOVC_USERNAME}" \
  --password "${GOVC_PASSWORD}" \
  --provider-insecure-skip-tls \
  -n "${NS}"
```

## oVirt / RHV

```bash
oc mtv create provider --name ovirt --type ovirt \
  --url "${RHV_URL}" \
  --username "${RHV_USERNAME}" \
  --password "${RHV_PASSWORD}" \
  --cacert "$(fetch_ca_cert "${RHV_URL}")" \
  -n "${NS}"
```

## OpenStack

```bash
oc mtv create provider --name openstack --type openstack \
  --url "${OSP_URL}" \
  --username "${OSP_USERNAME}" \
  --password "${OSP_PASSWORD}" \
  --provider-domain-name "${OSP_DOMAIN_NAME}" \
  --provider-project-name "${OSP_PROJECT_NAME}" \
  --provider-region-name "${OSP_REGION_NAME}" \
  --cacert "$(fetch_ca_cert "${OSP_URL}")" \
  -n "${NS}"
```

## OVA

```bash
oc mtv create provider --name ova --type ova \
  --url "${OVA_URL}" \
  -n "${NS}"
```

## Remote OpenShift (source)

```bash
oc mtv create provider --name source-ocp --type openshift \
  --url "${SOURCE_OCP_URL}" \
  --provider-token "${SOURCE_OCP_TOKEN}" \
  --provider-insecure-skip-tls \
  -n "${NS}"
```

The remote OpenShift cluster is typically the **source** (where VMs live). Use the
`SOURCE_` prefix for its env vars to distinguish it from the local cluster running MTV.

## HyperV

```bash
oc mtv create provider --name hyperv --type hyperv \
  --url "${HV_URL}" \
  --username "${HV_USERNAME}" \
  --password "${HV_PASSWORD}" \
  --smb-url "${HV_SMB_URL}" \
  --provider-insecure-skip-tls \
  -n "${NS}"
```

## EC2

```bash
oc mtv create provider --name ec2 --type ec2 \
  --ec2-region "${EC2_REGION}" \
  --target-access-key-id "${EC2_ACCESS_KEY_ID}" \
  --target-secret-access-key "${EC2_SECRET_ACCESS_KEY}" \
  --target-az "${EC2_TARGET_AZ}" \
  --target-region "${EC2_TARGET_REGION}" \
  -n "${NS}"
```

## OpenShift (local cluster)

```bash
oc mtv create provider --name host --type openshift -n "${NS}"
```

The local OpenShift provider auto-detects the current cluster and needs no URL or
token. It is typically the target but can serve as the source when a remote
OpenShift is the target.
