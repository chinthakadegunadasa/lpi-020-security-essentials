# Chapter 13: Storage Architecture, Volume Management, and File Systems
## 13.1 Enterprise Storage Models and Linux Storage Architecture
Linux storage management spans physical drives, block device layers, logical volume abstractions, and high-performance file systems. A structured approach ensures system reliability, scalability, and high performance for enterprise workloads.
```
                  ENTERPRISE LINUX STORAGE ARCHITECTURE

   +------------------------------------------------------------------+
   |                       User / Application                         |
   |           (Reads/Writes via System Calls: VFS Layer)            |
   +------------------------------------------------------------------+
                                    |
                                    v
   +------------------------------------------------------------------+
   |                       File System Layer                          |
   |              (ext4, XFS, Btrfs - Files & Directories)            |
   +------------------------------------------------------------------+
                                    |
                                    v
   +------------------------------------------------------------------+
   |                     Logical Volume Manager                       |
   |       Logical Volumes (LV) <--- Volume Groups (VG)              |
   +------------------------------------------------------------------+
                                    |
                                    v
   +------------------------------------------------------------------+
   |                   Physical Volume Layer (PV)                     |
   |       Physical Partitions / Disks (e.g., /dev/sdb1, /dev/nvme0n1)|
   +------------------------------------------------------------------+
                                    |
                                    v
   +------------------------------------------------------------------+
   |                     Physical Block Devices                       |
   |               (SATA, SAS, NVMe SSD, SAN LUNs)                    |
   +------------------------------------------------------------------+

```
### Partitioning Schemes: MBR vs. GPT
| Feature | MBR (Master Boot Record) | GPT (GUID Partition Table) |
|---|---|---|
| **Max Disk Size** | 2 TB | 9.4 ZB (Zettabytes) |
| **Max Primary Partitions** | 4 (or 3 Primary + 1 Extended) | 128 (default in Linux) |
| **Partition Table Backup** | None (Single point of failure) | Primary & Backup CRC32 Table |
| **Boot Mode Support** | Legacy BIOS | UEFI / Modern Firmware |
## 13.2 File System Comparison & Enterprise Use Cases
Enterprise Linux environments utilize distinct file systems depending on structural needs, scalability, and recovery requirements.
| File System | Max File Size | Key Features & Architecture | Enterprise Use Cases |
|---|---|---|---|
| **ext4** | 16 TB | Block allocation using extents, backwards compatible with ext2/ext3, journaling. | Default for general-purpose Linux servers, web servers, boot partitions. |
| **XFS** | 8 EB | 64-bit high-performance journaling, dynamic inode allocation, parallel I/O allocations. | Enterprise database servers, big data storage, default file system on RHEL/SLES. |
| **Btrfs** | 16 EB | Copy-on-Write (CoW), integrated volume management, snapshots, self-healing data checksums. | Virtualization host storage, modern enterprise backups, container rootfs storage. |
## 13.3 Logical Volume Manager (LVM) Core Concepts
LVM introduces a flexible layer of abstraction between physical storage devices and file systems.
 * **Physical Volume (PV):** Raw block devices or partitions initialized for LVM (e.g., /dev/sdb1).
 * **Volume Group (VG):** A storage pool created by aggregating one or more Physical Volumes.
 * **Logical Volume (LV):** A virtual partition carved out of a Volume Group upon which file systems are created.
 * **Physical Extent (PE):** The smallest allocable chunk of storage within a VG (typically 4 MB by default).
```
   Physical Disks            Physical Volumes            Volume Group            Logical Volumes
+------------------+       +------------------+        +--------------+        +-----------------+
| /dev/sdb (50GB)  |  -->  |   pvcreate /dev/sdb  |  \     |              |  -->   |  /dev/vg0/lv_app|
+------------------+       +------------------+   \    |  vgcreate    |        +-----------------+
                                                   +-> |    vg0       |
+------------------+       +------------------+   /    |  (100 GB)    |        +-----------------+
| /dev/sdc (50GB)  |  -->  |   pvcreate /dev/sdc  |  /     |              |  -->   |  /dev/vg0/lv_data
+------------------+       +------------------+        +--------------+        +-----------------+

```
## 13.4 Persistence and Mount Management (/etc/fstab)
To ensure file systems mount automatically across system reboots, entries are declared in /etc/fstab. Modern best practices mandate using UUIDs (Universally Unique Identifiers) instead of raw device nodes (e.g., /dev/sdb1) to prevent mount failures if drive enumeration changes.
### Syntax Structure of /etc/fstab
```
# <file system / UUID>                     <mount point>  <type>  <options>       <dump>  <fsck>
UUID=a1b2c3d4-e5f6-7890-abcd-1234567890ab  /mnt/data      xfs     defaults,noatime  0       2

```
## 13.5 Hands-On Lab Framework: Partitioning, LVM Allocation, and Dynamic File System Expansion
### Lab Objectives
 1. Inspect available block devices using low-level tools.
 2. Partition storage using parted (GPT table).
 3. Create an LVM topology (PV, VG, and LV).
 4. Format with XFS and configure persistent mounting via UUID in /etc/fstab.
 5. On-the-fly expansion of an LVM volume and live file system growth.
### Step 1: Storage Device Inspection
 1. List all block storage devices and existing topologies:
   ```bash
   lsblk -f
   
   ```
 2. Inspect disk architecture and geometry using fdisk:
   ```bash
   sudo fdisk -l
   
   ```
 3. Identify UUIDs of existing block devices:
   ```bash
   sudo blkid
   
   ```
### Step 2: Create GPT Partitions with parted
Assume /dev/sdb (or a loop device) is available for configuration.
 1. Create a loopback block device for demonstration if physical disks are not present:
   ```bash
   sudo dd if=/dev/zero of=/var/tmp/disk1.img bs=1M count=2048
   sudo losetup -fP /var/tmp/disk1.img
   # Determine assigned loop device (e.g., /dev/loop0)
   LOOP_DEV=$(losetup -j /var/tmp/disk1.img | cut -d: -f1)
   echo "Using target device: $LOOP_DEV"
   
   ```
 2. Initialize a GPT partition table on the device:
   ```bash
   sudo parted -s "$LOOP_DEV" mklabel gpt
   
   ```
 3. Create two 1 GB partitions set for LVM flag:
   ```bash
   sudo parted -s "$LOOP_DEV" mkpart primary 1MiB 1000MiB
   sudo parted -s "$LOOP_DEV" mkpart primary 1001MiB 2000MiB
   sudo parted -s "$LOOP_DEV" set 1 lvm on
   sudo parted -s "$LOOP_DEV" set 2 lvm on
   
   ```
 4. Verify partition structure:
   ```bash
   sudo parted "$LOOP_DEV" print
   
   ```
### Step 3: Build LVM Storage Topology
 1. Initialize Physical Volumes (PVs):
   ```bash
   sudo pvcreate "${LOOP_DEV}p1" "${LOOP_DEV}p2"
   sudo pvs
   
   ```
 2. Create a Volume Group named vg_enterprise:
   ```bash
   sudo vgcreate vg_enterprise "${LOOP_DEV}p1" "${LOOP_DEV}p2"
   sudo vgs
   
   ```
 3. Allocate a 1.2 GB Logical Volume named lv_data:
   ```bash
   sudo lvcreate -L 1.2G -n lv_data vg_enterprise
   sudo lvs
   
   ```
### Step 4: Format and Persistently Mount File System
 1. Format the logical volume with the XFS file system:
   ```bash
   sudo mkfs.xfs /dev/vg_enterprise/lv_data
   
   ```
 2. Create target mount directory:
   ```bash
   sudo mkdir -p /mnt/enterprise_data
   
   ```
 3. Extract the UUID of the newly created logical volume:
   ```bash
   FS_UUID=$(sudo blkid -s UUID -o value /dev/vg_enterprise/lv_data)
   echo "Target UUID: $FS_UUID"
   
   ```
 4. Add persistent entry to /etc/fstab:
   ```bash
   echo "UUID=$FS_UUID /mnt/enterprise_data xfs defaults 0 2" | sudo tee -a /etc/fstab
   
   ```
 5. Test mounting using /etc/fstab:
   ```bash
   sudo mount -a
   df -hT /mnt/enterprise_data
   
   ```
### Step 5: Live Expansion of LVM Volume and XFS File System
 1. Add a secondary loop device to simulate adding physical storage to the server:
   ```bash
   sudo dd if=/dev/zero of=/var/tmp/disk2.img bs=1M count=1024
   sudo losetup -fP /var/tmp/disk2.img
   LOOP_DEV2=$(losetup -j /var/tmp/disk2.img | cut -d: -f1)
   
   ```
 2. Initialize new device as PV and extend vg_enterprise:
   ```bash
   sudo pvcreate "$LOOP_DEV2"
   sudo vgextend vg_enterprise "$LOOP_DEV2"
   sudo vgs
   
   ```
 3. Expand lv_data dynamically by adding 500 MB:
   ```bash
   sudo lvextend -L +500M /dev/vg_enterprise/lv_data
   
   ```
 4. Extend the live XFS file system on-the-fly (no unmounting required):
   ```bash
   sudo xfs_growfs /mnt/enterprise_data
   df -hT /mnt/enterprise_data
   
   ```
### Step 6: Clean Up Lab Environment
 1. Unmount directory and clean up persistent configuration:
   ```bash
   sudo umount /mnt/enterprise_data
   sudo sed -i "/$FS_UUID/d" /etc/fstab
   
   ```
 2. Tear down LVM structure and loop devices:
   ```bash
   sudo lvremove -f /dev/vg_enterprise/lv_data
   sudo vgremove vg_enterprise
   sudo pvremove "${LOOP_DEV}p1" "${LOOP_DEV}p2" "$LOOP_DEV2"
   sudo losetup -d "$LOOP_DEV" "$LOOP_DEV2"
   sudo rm -f /var/tmp/disk1.img /var/tmp/disk2.img /mnt/enterprise_data
   
   ```
## 13.6 Chapter Review Questions
 1. Which partition table standard supports disk capacities greater than 2 TB and provides redudant primary/backup routing tables?
   * A) MBR
   * B) GPT
   * C) VFAT
   * D) EXT3
 2. In the Logical Volume Manager (LVM) framework, what component represents the aggregated storage pool formed by combining individual storage drives?
   * A) Physical Volume (PV)
   * B) Logical Volume (LV)
   * C) Volume Group (VG)
   * D) Extent Allocation Table (EAT)
 3. Which utility is specifically required to resize and expand an online XFS file system after increasing its underlying Logical Volume capacity?
   * A) resize2fs
   * B) xfs_growfs
   * C) fsck.xfs
   * D) gparted
 4. Why is using device node identifiers like /dev/sdb1 in /etc/fstab considered unsafe for production Linux servers?
   * A) Device names change dynamically based on drive discovery order during boot.
   * B) Device nodes do not support XFS or Btrfs file systems.
   * C) Systemd restricts standard path declarations in /etc/fstab.
   * D) Direct device nodes require unencrypted partition tables.
## 13.7 Key Terms Glossary
 * **Btrfs:** A modern Copy-on-Write (CoW) file system for Linux designed for fault tolerance, repair, and easy administration.
 * **ext4:** Extended File System version 4, a standard, highly reliable Linux journaling file system.
 * **Logical Volume (LV):** A virtual block device created within an LVM Volume Group that holds file systems.
 * **Logical Volume Manager (LVM):** A storage management abstraction layer providing flexible disk layout capabilities.
 * **MBR (Master Boot Record):** Legacy 32-bit partition table scheme located in the first sector of a storage disk.
 * **Physical Extent (PE):** The smallest fixed-size block of memory allocated within an LVM Volume Group.
 * **UUID (Universally Unique Identifier):** A 128-bit number used to uniquely identify file systems across system reboots.
 * **XFS:** A high-performance 64-bit journaling file system designed for high I/O parallelism and large scale workloads.
 
