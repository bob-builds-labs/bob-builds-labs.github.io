# DDVE Helm Chart - Quick Start

## Quick Reference

### Step 1: Pull and Extract the Chart
```bash
helm pull oci://quay.io/delldps/charts/ddve --version 8.9.0-0
tar -xzf ddve-8.9.0-0.tgz
```

### Step 2: View Default Values
```bash
# View from Quay (no need to extract)
helm show values oci://quay.io/delldps/charts/ddve --version 8.9.0-0

# Or view from extracted chart
cat ddve/values.yaml
```

### Step 3: Create Golden Resources (One-Time)
```bash
# Option 1: From parent directory (recommended - no cd needed)
helm install golden-resources oci://quay.io/delldps/charts/ddve --version 8.9.0-0 -f ddve/values-golden.yaml

# Option 2: From ddve directory
cd ddve
helm install golden-resources oci://quay.io/delldps/charts/ddve --version 8.9.0-0 -f values-golden.yaml
cd ..
```

### Step 4: Monitor the Import
```bash
# Watch the DataVolume import progress
oc get datavolume ddve-8.9.0.0-1263812-pvc -n openshift-virtualization-os-images -w

# Check the phase (should show "Succeeded" when complete)
oc get datavolume ddve-8.9.0.0-1263812-pvc -n openshift-virtualization-os-images
```

### Step 5: Deploy VM
```bash
# Option 1: From parent directory (recommended)
helm install my-ddve oci://quay.io/delldps/charts/ddve --version 8.9.0-0 -f ddve/values-example-basic.yaml -n ddve --create-namespace

# Option 2: From ddve directory
cd ddve
helm install my-ddve oci://quay.io/delldps/charts/ddve --version 8.9.0-0 -f values-example-basic.yaml -n ddve --create-namespace
cd ..
```

### Step 6: Start VM
```bash
virtctl start my-ddve -n ddve
```

### Step 7: Check Status
```bash
oc get vm -n ddve
oc get vmi -n ddve
oc get pvc -n ddve
```

## Cleanup Failed Import

If the golden import fails, clean up the failed resources before retrying:

```bash
# Delete the Helm release
helm uninstall golden-resources -n openshift-virtualization-os-images

# Delete the failed DataVolume
oc delete datavolume ddve-8.9.0.0-1263812-pvc -n openshift-virtualization-os-images

# Delete the PVC if it was created
oc delete pvc ddve-8.9.0.0-1263812-pvc -n openshift-virtualization-os-images

# Delete the DataSource if it was created
oc delete datasource ddve-8.9.0.0-1263812 -n openshift-virtualization-os-images

# Check for any stuck importer pods
oc get pods -n cdi | grep importer
# Delete if needed
oc delete pod <importer-pod> -n cdi
```

## Example Configurations

| File | Description | Use Case |
|------|-------------|----------|
| `ddve/values-example-basic.yaml` | 1 disk, 1 network, 8TB | Simple deployment |
| `ddve/values-example-multinic.yaml` | 4 disks, 3 networks, 32TB | Multi-network setup |
| `ddve/values-example-high-capacity.yaml` | 8 disks, 2 networks, 96TB | High capacity |
| `ddve/values-golden.yaml` | Golden resources only | One-time setup |

## Common Commands

### Get Chart Info
```bash
helm show chart oci://quay.io/delldps/charts/ddve --version 8.9.0-0
```

### Get README
```bash
helm show readme oci://quay.io/delldps/charts/ddve --version 8.9.0-0
```

### Upgrade
```bash
# From parent directory
helm upgrade my-ddve oci://quay.io/delldps/charts/ddve --version <new-version> -f ddve/my-values.yaml -n ddve
```

### Uninstall
```bash
helm uninstall my-ddve -n ddve
```

## Important Notes

- **Golden resources** (PVC/DataSource) are created once per version in `openshift-virtualization-os-images`
- **VMs** are deployed in separate namespaces and reference the shared golden resources
- **First deployment**: Use `ddve/values-golden.yaml` to create golden resources
- **Subsequent deployments**: Use example files, golden resources already exist
- **File paths**: All commands work from parent directory using `ddve/` prefix. No need to `cd ddve`.
- **Single VM per release**: By default, each Helm release deploys one VM. To deploy multiple VMs in the same namespace, use the multi-VM example files (see "Deploy Multiple VMs" section below).

## Troubleshooting

### Chart Not Found
```bash
helm pull oci://quay.io/delldps/charts/ddve --version 8.9.0-0
```

### DataSource Not Found
```bash
oc get datasource -n openshift-virtualization-os-images
```

### VM Won't Start
```bash
oc get datavolume my-ddve-rootdisk -n ddve
oc get datasource ddve-8.9.0.0-1263812 -n openshift-virtualization-os-images
```

### Import Failed
If the CDI import fails:
1. Check the importer pod logs: `oc logs <importer-pod> -n cdi`
2. Verify the image digest is correct: `podman pull quay.io/delldps/ddve-appliance@<digest>`
3. Clean up failed resources (see "Cleanup Failed Import" section above)
4. Retry the installation

### Import Stuck
If the import is stuck:
1. Watch the DataVolume: `oc get datavolume ddve-8.9.0.0-1263812-pvc -n openshift-virtualization-os-images -w`
2. Check the phase - it should progress from "Pending" to "Importing" to "Succeeded"
3. If stuck in "Importing" for > 30 minutes, check the importer pod logs
4. Clean up and retry if needed

### Namespace Already Has a Release
If you get "invalid ownership metadata" error:
```bash
# Error: Namespace "ddve" in namespace "" exists and cannot be imported into the current release
# This means the namespace already has a Helm release

# Option 1: Uninstall the existing release
helm uninstall my-ddve -n ddve

# Option 2: Use a different namespace
helm install my-ddve1 oci://quay.io/delldps/charts/ddve --version 8.9.0-0 -f ddve/values-example-basic.yaml -n ddve1 --create-namespace

# Option 3: Upgrade the existing release
helm upgrade my-ddve oci://quay.io/delldps/charts/ddve --version 8.9.0-0 -f ddve/values-example-basic.yaml -n ddve
```

## Deploy Multiple VMs in the Same Namespace

The chart supports deploying multiple DDVE VMs in a single Helm release using the `vms` list configuration.

### Deploy Multiple VMs

```bash
# Deploy 2 VMs in the same namespace
helm install my-ddve oci://quay.io/delldps/charts/ddve --version 8.9.0-0 -f ddve/values-example-multivm.yaml -n ddve --create-namespace

# Deploy 3 VMs in the same namespace
helm install my-ddve oci://quay.io/delldps/charts/ddve --version 8.9.0-0 -f ddve/values-example-multivm-3.yaml -n ddve --create-namespace
```

### Add VMs Over Time

You can add VMs to an existing deployment by upgrading with an updated values file:

```bash
# Initial deployment with 1 VM (using single VM mode)
helm install my-ddve oci://quay.io/delldps/charts/ddve --version 8.9.0-0 -f ddve/values-example-basic.yaml -n ddve --create-namespace

# Later, upgrade to add a second VM (switch to multi-VM mode)
helm upgrade my-ddve oci://quay.io/delldps/charts/ddve --version 8.9.0-0 -f ddve/values-example-multivm.yaml -n ddve
```

Helm will create the new VM resources without modifying the existing VM.

### Start Multiple VMs

```bash
# Start all VMs
virtctl start ddve-1 -n ddve
virtctl start ddve-2 -n ddve

# Or start specific VMs
virtctl start ddve-1 -n ddve
```

### Multi-VM Configuration

The multi-VM example files show how to configure multiple VMs:

- `values-example-multivm.yaml` - 2 VMs with different configurations
- `values-example-multivm-3.yaml` - 3 VMs with different configurations

Each VM in the list can have:
- Unique name
- Different license tier (CPU/memory)
- Different storage configuration
- Different network configuration

### Single VM vs Multi-VM Mode

**Single VM mode** (backward compatible):
- Uses `vm.name` in values.yaml
- Deploys one VM per release
- Existing deployments continue to work

**Multi-VM mode**:
- Uses `vms` list in values.yaml
- Deploys multiple VMs per release
- Each VM has unique resources (PVCs, DataVolumes, VirtualMachine)
- Namespace labels aggregate all networks from all VMs

## Full Documentation

For detailed instructions, see [USER_GUIDE.md](USER_GUIDE.md)
