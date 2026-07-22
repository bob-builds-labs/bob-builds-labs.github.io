
# Dell PowerProtect DDVE — In-Cluster Deployment Plan
## OpenShift Virtualization / KubeVirt

> **Scope:** In-cluster usage only. DDVE is consumed by backup clients running inside the same OpenShift cluster (e.g. PowerProtect Data Manager, Velero with DD Boost plugin, or application pods using NFS/DD Boost). No external replication or external client access is covered in this plan.

> **Support notice:** Dell does not officially certify DDVE on OpenShift Virtualization. This plan is technically sound and translates the official KVM procedure into KubeVirt primitives, but should be treated as best-effort until Dell adds OCP Virt to its support matrix.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│ OpenShift Cluster                                           │
│                                                             │
│  ┌─────────────────────┐      ClusterIP / LB Service        │
│  │   DDVE VM (KubeVirt)│◄────────────────────────────────┐  │
│  │   pod network NIC   │   port 2052 (DD Boost)          │  │
│  │   (masquerade NAT)  │   port 2049 (NFS)               │  │
│  └─────────────────────┘   port 443/3009 (DDMGMT)        │  │
│           │ RWO Block PVCs                                │  │
│           │ (root + nvram + 4×2Ti data)          ┌───────┴──┐│
│  ┌────────▼────────────┐                         │ Backup   ││
│  │  ODF / Block        │                         │ Clients  ││
│  │  StorageClass       │                         │ (pods)   ││
│  └─────────────────────┘                         └──────────┘│
└─────────────────────────────────────────────────────────────┘
```

* **Networking:** default OCP pod network (masquerade). DDVE is exposed inside the cluster via a `ClusterIP` Service. No Multus bridge NICs required.
* **Storage:** RWO Block PVCs on a block-capable StorageClass (e.g. ODF Ceph RBD). One PVC per disk — no shared/RWX needed.
* **Data disks:** max 2 TiB each; 4 disks = 8 TiB raw (≈ effective capacity after dedup/compression varies per workload).
* **No external replication** configured in this plan.

---

## Sizing — 8 TB Configuration

| Parameter | Value |
|---|---|
| Target raw capacity | 8 TiB |
| Data disks | 4 × 2 TiB (2 TiB ceiling per disk) |
| vCPU | 2 |
| Memory | 8 GiB (requests = limits, no overcommit) |
| Root disk | 350 GiB (v8.8) / 250 GiB (v7.13) |
| vNVRAM disk | 10 GiB |
| Total PVC count | 6 (1 root + 1 nvram + 4 data) |

### Why 2 TiB per disk (not larger)

* Aligns with Dell's KVM reference scripts which keep individual disks operationally manageable
* Avoids long sparse-allocation times on Ceph RBD at provisioning
* Allows incremental capacity expansion later (add more 2 TiB disks) without resizing existing PVCs
* Keeps individual disk rebuild times bounded in case of storage failure

### Why RWO Block (not RWX, not Filesystem mode)

* DDVE is a single-node appliance — only one writer ever mounts these disks; RWX provides no benefit and adds Ceph RBD locking overhead
* Block `volumeMode` is mandatory — DDVE manages its own on-disk layout; a Filesystem-mode PVC would add a filesystem layer that corrupts DDVE's internal structures
* RWO Block gives the best sequential and random I/O performance for dedup/compression workloads

---

## Prerequisites

* OpenShift Container Platform 4.12+ with OpenShift Virtualization operator installed and healthy
* CDI (Containerized Data Importer) enabled — included with OpenShift Virtualization
* `virtctl` CLI installed and authenticated
* `oc` CLI with namespace-admin or cluster-admin rights
* A block-capable StorageClass (e.g. `ocs-storagecluster-ceph-rbd-virtualization`) — check with `oc get sc`
* DDVE KVM `.tar.gz` downloaded from [Dell Support](https://www.dell.com/support/home/en-us/product-support/product/dd-virtual-edition/overview) and extracted on a Linux workstation

---

## Phase 1 — Namespace

```yaml
# 00-namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ddve
```

```bash
oc apply -f 00-namespace.yaml
```

---

## Phase 2 — Import Root Disk Image

No conversion is needed. CDI converts `.qcow2` to raw block format automatically on upload.

```bash
virtctl image-upload dv ddve-rootdisk \
  --namespace ddve \
  --size=350Gi \
  --image-path=${DDVE_IMAGE} \
  --storage-class=ocs-storagecluster-ceph-rbd-virtualization \
  --volume-mode=block \
  --access-mode=ReadWriteOnce \
  --wait-secs=600
```

Monitor until `PHASE: Succeeded`:
```bash
oc get datavolume ddve-rootdisk -n ddve -w
```

---

## Phase 3 — Storage PVCs

### vNVRAM disk (SCSI 0:1)

DDVE requires a dedicated NVRAM simulation disk. It must be a separate PVC — do not use the root disk for this purpose.

```yaml
# 04-pvc-nvram.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ddve-nvram
  namespace: ddve
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Block          # MUST be Block — never Filesystem
  storageClassName: ocs-storagecluster-ceph-rbd-virtualization
  resources:
    requests:
      storage: 10Gi
```

### Data disks (SCSI 0:2 – 0:5) — 4 × 2 TiB

* Maximum **2 TiB per disk** — do not exceed this ceiling
* All disks must be the **same size** — DDVE expects a homogeneous disk set; mixing sizes is not supported
* Use `ReadWriteOnce` — DDVE never co-mounts a data disk from a second host; RWX is unnecessary and degrades performance
* Use `volumeMode: Block` — mandatory; Filesystem mode must not be used for data disks

```yaml
# 05-pvc-datadisks.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ddve-data-1    # repeat for ddve-data-2, ddve-data-3, ddve-data-4
  namespace: ddve
spec:
  accessModes:
    - ReadWriteOnce    # RWO — single writer, no concurrent mount possible
  volumeMode: Block    # Block mandatory — Filesystem mode corrupts DDVE internal structures
  storageClassName: ocs-storagecluster-ceph-rbd-virtualization
  resources:
    requests:
      storage: 2Ti     # 2 TiB ceiling per disk — do not exceed
```

Apply all four:
```bash
for i in 1 2 3 4; do
  sed "s/ddve-data-1/ddve-data-${i}/" 05-pvc-datadisks.yaml | oc apply -f -
done
```

#### Capacity expansion

To grow beyond 8 TiB later, add more 2 TiB PVCs and hot-plug them:

```bash
virtctl addvolume ddve \
  --volume-name=ddve-data-5 \
  --persist \
  -n ddve
```

Do **not** resize existing PVCs — expanding in-place is not supported by DDVE's disk management.

---

## Phase 4 — VirtualMachine

```yaml
# 06-vm-ddve.yaml
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: ddve
  namespace: ddve
  labels:
    app: ddve
spec:
  running: false
  template:
    metadata:
      labels:
        kubevirt.io/domain: ddve
    spec:
      domain:
        # ── CPU (no overcommit) ──────────────────────────────
        cpu:
          cores: 2
          sockets: 1
          threads: 1
        resources:
          requests:
            memory: 8Gi
          limits:
            memory: 8Gi        # requests == limits — overcommit not supported by Dell
        memory:
          hugepages:
            pageSize: 2Mi      # recommended for DDVE dedup/compression workloads

        # ── Machine type ────────────────────────────────────
        machine:
          type: pc-q35-rhel9.4.0   # adjust to your OCP version; q35 required for SCSI

        # ── Firmware (UEFI) ─────────────────────────────────
        firmware:
          bootloader:
            efi:
              secureBoot: false

        # ── Devices ─────────────────────────────────────────
        devices:
          disks:
            - name: rootdisk       # SCSI 0:0 — boot disk
              bootOrder: 1
              disk:
                bus: scsi
            - name: nvramdisk      # SCSI 0:1 — vNVRAM
              disk:
                bus: scsi
            - name: datadisk1      # SCSI 0:2
              disk:
                bus: scsi
#            - name: datadisk2      # SCSI 0:3
#              disk:
#                bus: scsi
#            - name: datadisk3      # SCSI 0:4
#              disk:
#                bus: scsi
#            - name: datadisk4      # SCSI 0:5
#              disk:
#                bus: scsi
          # ── Single NIC — default pod network (masquerade) ──
          interfaces:
            - name: default
              masquerade: {}       # in-cluster only; NAT is sufficient for pod-to-pod traffic

      # ── Networks ──────────────────────────────────────────
      networks:
        - name: default
          pod: {}                  # OCP pod network — no Multus required

      # ── Volumes ───────────────────────────────────────────
      volumes:
        - name: rootdisk
          persistentVolumeClaim:
            claimName: ddve-rootdisk
        - name: nvramdisk
          persistentVolumeClaim:
            claimName: ddve-nvram
        - name: datadisk1
          persistentVolumeClaim:
            claimName: ddve-data-1
#        - name: datadisk2
#          persistentVolumeClaim:
#            claimName: ddve-data-2
#        - name: datadisk3
#          persistentVolumeClaim:
#            claimName: ddve-data-3
#        - name: datadisk4
#          persistentVolumeClaim:
#            claimName: ddve-data-4
```

```bash
oc apply -f 06-vm-ddve.yaml
```

---

## Phase 5 — Service (in-cluster access)

A `ClusterIP` Service exposes DDVE to other pods inside the cluster. All backup protocols are reachable within the cluster without leaving the SDN.

```yaml
# 07-service-ddve.yaml
apiVersion: v1
kind: Service
metadata:
  name: ddve
  namespace: ddve
spec:
  selector:
    kubevirt.io/domain: ddve
  type: ClusterIP
  ports:
    - name: ddboost
      port: 2052
      targetPort: 2052
      protocol: TCP
    - name: nfs
      port: 2049
      targetPort: 2049
      protocol: TCP
    - name: ddmgmt-https
      port: 443
      targetPort: 443
      protocol: TCP
    - name: ddmgmt-gui
      port: 3009
      targetPort: 3009
      protocol: TCP
    - name: ssh
      port: 22
      targetPort: 22
      protocol: TCP
```

```bash
oc apply -f 07-service-ddve.yaml
```

Backup clients in any namespace reach DDVE at:
```
ddve.ddve.svc.cluster.local
```

---

## Phase 6 — Start and Initial Configuration

### Start the VM

```bash
virtctl start ddve -n ddve
```

### Connect to console

```bash
virtctl console ddve -n ddve
```

### DDOS initial network configuration

With masquerade NAT, DDVE receives its pod IP via DHCP automatically on `ethV0`. Verify connectivity:

```
net show config
```

If DHCP did not assign an address, set it manually:
```
net set interface ethV0 ipaddr <POD-IP> netmask <MASK>
net config defaultroute <GW>
net enable interface ethV0
```

### DDOS initial configuration wizard

```bash
system set hostname ddve.ddve.svc.cluster.local
system set timezone Europe/Berlin          # adjust to your timezone
user change password admin
filesys create
filesys enable
```

### Access DD System Manager (GUI)

From within the cluster (e.g. an `oc port-forward`):
```bash
oc port-forward svc/ddve 3009:3009 -n ddve
```
Then open: `https://localhost:3009`

Or from any pod inside the cluster:
```
https://ddve.ddve.svc.cluster.local:3009
```

### Apply license

```bash
system show serialno    # note the system ID
```

Apply the capacity license key in DD System Manager under **Administration → Licenses**.

---

## Phase 7 — Configure Backup Client Access

### DD Boost (recommended for PowerProtect / Networker)

On the DDOS CLI:
```bash
ddboost enable
ddboost user assign <username>
ddboost storage-unit create <storage-unit-name>
```

Backup client connection string:
```
ddve.ddve.svc.cluster.local  port 2052
```

### NFS (for pod-based clients)

```bash
nfs enable
nfs add /data/col1/<storage-unit> <client-pod-cidr> (rw,no_root_squash)
```

Mount from a pod:
```
ddve.ddve.svc.cluster.local:/data/col1/<storage-unit>
```

---

## Deployment Checklist

* [ ] OCP 4.12+ with OpenShift Virtualization operator healthy
* [ ] CDI enabled; `virtctl` authenticated
* [ ] Block StorageClass confirmed (`oc get sc`)
* [ ] Namespace `ddve` created
* [ ] Root disk DataVolume uploaded and `PHASE: Succeeded`
* [ ] vNVRAM PVC created (10 Gi, Block, RWO)
* [ ] 4 × 2 TiB data PVCs created (Block, RWO, same size)
* [ ] VirtualMachine CR applied (single pod-network NIC, SCSI disks, 2 vCPU / 8 GiB, hugepages)
* [ ] VM started; console accessible
* [ ] DDOS initial config completed (hostname, filesys create, filesys enable)
* [ ] License applied
* [ ] ClusterIP Service applied; `ddve.ddve.svc.cluster.local` resolves from backup client pods
* [ ] DD Boost or NFS configured and connectivity verified from backup client
