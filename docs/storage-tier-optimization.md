# Architectural Update: Automated Storage Tiering & USB Backup Isolation

## 📌 Executive Summary
As the compute footprint of Proxmox environment expands its hosting with a multi-threaded recursive Technitium DNS topology, high-throughput 2.5 Gbps LanCache server, and stateful Splunk SIEM pipeline—the primary 1TB NVMe drive began handling dangerous blends of high-performance operating system execution and bulky backup blocks. 

To prevent disk exhaustion on the primary partition, mitigate high I/O wait delay, and isolate the system blast radius, we executed a physical hardware re-allocation. This update details the integration of an **internal 1TB 2.5" mechanical HDD (`storage-internal`)** and an **external 1TB WDC USB Drive (`storage-external`)** into Proxmox using enterprise-grade storage separation principles.

---

## 🏗️ Optimized Storage Topology

The hypervisor cluster has been transitioned to a dedicated, multi-tiered data framework:


```

[ 1TB NVMe SSD ]     ⚡ Performance Tier -> Live VM Virtual Disks & LXC Filesystems
│                                    (local-lvm: images, rootdir)
▼
[ 1TB 2.5" HDD ]     📂 Capacity Tier    -> Unburdened Deployment Libraries
│                                    (storage-internal: iso, vztmpl, snippets)
▼
[ 1TB WDC USB ]      🛡️ Backup Vault     -> Isolated Multi-Generational Snapshots
(storage-external: backup)

```

---

## ⚙️ Technical Implementation Details

### 1. File System Mount Correction (`/etc/fstab`)
To resolve Proxmox `500 directory expected to be a mount point` activation failures, block storage targets were cross-referenced using `lsblk` and tied natively to their unique filesystem hardware markers. 
The external USB drive includes the `nofail` flag to prevent hypervisor kernel panics or boot hanging if the cable interface is disconnected during a power cycle.

```file
# /etc/fstab configuration entries
UUID=your-internal-hdd-uuid-here  /mnt/pve/storage-internal  ext4  defaults  0  2
UUID=your-wdc-usb-drive-uuid-here /mnt/pve/storage-external  ext4  defaults,nofail  0  2

```

### 2. Proxmox Storage Layer Allocation (`/etc/pve/storage.cfg`)

Content definitions were explicitly restricted at the hypervisor configuration layer. By dropping `rootdir` and `images` permissions from mechanical paths, Proxmox completely filters these drives out from default allocation dropdown menus, preventing accidental slow storage assignments.

```pve-design
dir: local
        path /var/lib/vz
        content iso,vztmpl,import

lvmthin: local-lvm
        thinpool data
        vgname pve
        content images,rootdir

dir: storage-internal
        path /mnt/pve/storage-internal
        content iso,vztmpl,snippets
        is_mountpoint 1
        nodes pve
        shared 0

dir: storage-external
        path /mnt/pve/storage-external
        content backup
        is_mountpoint 1
        nodes pve
        shared 0

```

---

## 🛡️ Operational Safeguards Engineered

* **NVMe Protection Latch (`is_mountpoint 1`):** Enabling this latch commands the storage engine to explicitly check the Linux mounting tables before initiating operations. If the external USB drive drops offline, backup tasks immediately fail-safe and abort, rather than silently writing backup fragments into the empty folder on the core 100GB NVMe root (`/`) filesystem.
* **NVMe Backup Blacklist:** The `VZDump backup file` flag was completely stripped from the `local` drive profile. This guarantees that automated arrays cannot target the OS boot partition, completely neutralizing disk-space exhaustion hazards.
* **"Blast Radius" Separation:** Unplugging the `storage-external` vault physically air-gaps cold disaster recovery data streams from live network components, building an essential security wall against potential file corruption or catastrophic platform mishaps.

---

## 🔄 Retention (Prune) Strategy

A strict, multi-layered snapshot pruning configuration was deployed to keep storage-external utilization predictable over long lab testing windows:

* **Keep Last:** `3` (Ensures immediate rollback continuity)
* **Keep Daily:** `7` (A full rolling week of point-in-time recovery histories)
* **Keep Weekly:** `4` (One complete calendar month of backup validation)

---
