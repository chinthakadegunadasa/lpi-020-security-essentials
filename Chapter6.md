# Chapter 6: Storage, Disk, and File System Encryption
## Objective Overview: LPI 022.4 Storage & Disk Encryption
| Field | Details |
|---|---|
| **Objective Code** | 022.4 |
| **Weight** | 3 |
| **Description** | Understand storage security concepts, block-level vs. file-system level encryption, key management, DM-Crypt, LUKS architecture, and eCryptfs/LUKS configuration and operation. |
| **Key Knowledge Areas** | Symmetric Ciphers for Disk Encryption (AES-XTS), LUKS Architecture (Header, Key Slots, Cipher Spec), DM-Crypt kernel subsystem, cryptsetup tool suite, Key Management & Passphrase rotation, File-system level encryption (eCryptfs, fscrypt), Swap Space Encryption, Key escrow and backup strategies. |
## 6.1 Disk Encryption Architecture & Mechanisms
Data-at-rest encryption protects data stored on block devices, laptops, removable media, and cloud storage volumes against physical theft, unauthorized access, or improper hardware disposal.
```
+-------------------------------------------------------------------------+
|                  STORAGE ENCRYPTION PARADIGMS                           |
+-------------------------------------------------------------------------+
                                     |
    +--------------------------------+--------------------------------+
    |                                                                 |
    v                                                                 v
+------------------------------------+              +------------------------------------+
|    BLOCK-LEVEL ENCRYPTION (LUKS)   |              |  FILE-SYSTEM ENCRYPTION (eCryptfs) |
+------------------------------------+              +------------------------------------+
|  [ User File System (ext4/xfs) ]   |              |  [ Encrypted Files / Directory ]   |
|                 |                  |              |                 |                  |
|                 v                  v              |                 v                  v
|  [ DM-Crypt Mapping (/dev/mapper) ]|              |  [ eCryptfs Stacking Layer ]       |
|                 |                  |              |                 |                  |
|                 v                  v              |                 v                  v
|  [ Raw Block Device (/dev/sdb1) ]  |              |  [ Underlying File System (ext4) ] |
+------------------------------------+              +------------------------------------+
| - Encrypts full block volume       |              | - Encrypts individual files/folders|
| - Hides metadata & directory tree  |              | - Filesystem metadata visible      |
| - Single master key via LUKS header|              | - Granular per-user key management |
+------------------------------------+              +------------------------------------+

```
### 1. Block-Level vs. File-System Encryption
#### Block-Level Encryption (LUKS / DM-Crypt)
Operates directly beneath the file system layer. Raw sector storage block data is transparently encrypted and decrypted before being passed to upper-layer file systems (ext4, xfs, btrfs).
 * **Advantages:** Complete metadata protection (file names, file sizes, directory structures, and permissions are completely hidden from raw storage reads).
 * **Disadvantages:** Storage space must be pre-allocated; resizing encrypted volumes requires specialized block manipulation.
#### File-System Level Encryption (eCryptfs / fscrypt)
Operates at or above the virtual file system (VFS) layer as a stacked file system or inline kernel feature.
 * **Advantages:** Works dynamically on existing directories without repartitioning; granular per-user file access control.
 * **Disadvantages:** File metadata (directory structure, file counts, file sizes, and timestamps) can remain visible to raw sector analysis depending on configuration.
### 2. Symmetric Ciphers & Modes for Disk Encryption
 * **AES-XTS Mode (aes-xts-plain64):** The standard cipher mode for disk encryption. **XTS** (XEX-based tweaked-codebook mode with ciphertext stealing) prevents pattern leaks when encrypting repeated disk sector blocks.
 * **Key Derivation Functions (KDF):** Transforms user passphrases into high-entropy cryptographic keys. Modern implementations use **Argon2id** or **PBKDF2** to mitigate brute-force and GPU-based dictionary attacks.
## 6.2 Linux Unified Key Setup (LUKS) Deep Dive
**LUKS (Linux Unified Key Setup)** is the standard specification for block-device encryption on Linux, providing a platform-independent, standardized binary header format.
```
+-------------------------------------------------------------------------+
|                          LUKS2 DISK STRUCTURE                           |
+-------------------------------------------------------------------------+
| [ LUKS Header ]                                                         |
|  - Cipher Specification (e.g., aes-xts-plain64)                         |
|  - Key Derivation Parameters (Argon2id, salt, iteration counts)         |
|  - Payload Offset Pointer                                               |
|  - Key Slots Array (Slots 0 - 7)                                        |
|    - Slot 0: Encrypted Master Key (Passphrase 1)                        |
|    - Slot 1: Encrypted Master Key (Passphrase 2 / Recovery Key)         |
|    - ...                                                                |
+-------------------------------------------------------------------------+
| [ Master Key ] (Kept in Kernel RAM during active mount)                 |
+-------------------------------------------------------------------------+
| [ Encrypted Payload Area ] (Actual disk blocks / File system data)     |
+-------------------------------------------------------------------------+

```
### LUKS Key Architecture
 1. **Master Key:** A single high-entropy random key that actually encrypts the raw storage payload blocks.
 2. **Key Slots:** LUKS provides up to 8 distinct key slots (or more in LUKS2). Each slot holds a copy of the *Master Key*, which is encrypted using a unique user passphrase or key file.
 3. **Passphrase Rotation:** Adding, modifying, or revoking a user passphrase simply re-encrypts the Master Key within that specific key slot. **The underlying payload data is never re-encrypted during passphrase changes.**
## 6.3 Swap Space Encryption & Security Best Practices
Leaving swap space unencrypted creates significant security risks on Linux systems. If system memory is swapped out to raw disk space, sensitive data—including plain-text credentials, SSH keys, and decrypted LUKS master keys—can be extracted from unencrypted swap partitions post-shutdown.
```
+-------------------------------------------------------------------------+
|                       SWAP SECURITY BEST PRACTICES                      |
+-------------------------------------------------------------------------+
|  1. Ephemeral Encrypted Swap:                                           |
|     Use cryptsetup with a fresh random key generated at every boot.     |
|                                                                         |
|  2. Hibernate Considerations:                                           |
|     If system hibernation (suspend-to-disk) is required, swap must    |
|     use a persistent LUKS key slot rather than a ephemeral key.         |
|                                                                         |
|  3. Header Backups:                                                     |
|     Back up the LUKS header block offline. If the LUKS header is       |
|     corrupted or overwritten, all data on the volume is permanently   |
|     unrecoverable.                                                      |
+-------------------------------------------------------------------------+

```
## 6.4 Practical Scenario: Securing Storage Volumes at CyberGate Systems
### Scenario Context
You are a Principal Storage & Security Systems Engineer at **CyberGate Services Private Limited**. A secondary 10GB storage volume (/dev/sdb) has been attached to an application node.
You must securely format this drive using **LUKS2** block-level encryption, configure automated mapping at system boot using keyfiles, establish an offline LUKS header backup, add a secondary recovery passphrase, and implement file-system level encryption for designated user home folders using **eCryptfs**.
```
  Unencrypted Volume (/dev/sdb)
             |
             v
   [ cryptsetup luksFormat ]
             |
             v
   [ Encrypted LUKS Container ] ---> Mapped to: /dev/mapper/secure_storage
                                                          |
                                                          v
                                              [ ext4 File System Installed ]

```
## 6.5 Hands-On Laboratory: Managing LUKS Block Encryption & eCryptfs
In this lab, you will format a block volume using LUKS2, manage LUKS key slots, set up automated volume mounting, back up cryptographic headers, and configure an eCryptfs private directory.
### Prerequisites
 * A Linux environment (Ubuntu/Debian or RHEL/Rocky Linux) with root privileges (sudo).
 * Tools installed: cryptsetup, eCryptfs-utils, and e2fsprogs.
### Step 1: Environment Setup and Loop Device Allocation
If you do not have a spare physical hard drive, create a 1GB virtual loopback block device for testing:
 1. Create a 1GB zeroed container file:
   ```bash
   sudo dd if=/dev/zero of=/var/tmp/encrypted_disk.img bs=1M count=1024 status=progress
   
   ```
 2. Associate the image file with a free loopback block device:
   ```bash
   sudo losetup -fP /var/tmp/encrypted_disk.img
   
   ```
 3. Identify the created loop device (e.g., /dev/loop0 or /dev/loop1):
   ```bash
   export LOOP_DEV=$(losetup -j /var/tmp/encrypted_disk.img | cut -d: -f1)
   echo "Allocated Block Device: ${LOOP_DEV}"
   
   ```
### Step 2: Formatting the Block Device with LUKS2 Encryption
 1. Format the block device using LUKS2 and AES-256-XTS cipher mode:
   ```bash
   sudo cryptsetup luksFormat --type luks2 --cipher aes-xts-plain64 --key-size 512 ${LOOP_DEV}
   
   ```
   * *Type YES in all capital letters when prompted, then supply a primary passphrase (e.g., PrimaryLUKS2026!).*
 2. Inspect the metadata header of the newly formatted LUKS device:
   ```bash
   sudo cryptsetup luksDump ${LOOP_DEV}
   
   ```
   * *Observe parameters: Version (LUKS2), Cipher (aes), Mode (xts-plain64), and Key Slot 0 (active).*
### Step 3: Opening, Formatting, and Mounting the Encrypted Volume
 1. Open the encrypted block device mapping layer:
   ```bash
   sudo cryptsetup open ${LOOP_DEV} secure_storage
   
   ```
   * *Enter the passphrase set in Step 2. This creates a virtual block mapping at /dev/mapper/secure_storage.*
 2. Verify that the device mapping exists:
   ```bash
   ls -l /dev/mapper/secure_storage
   
   ```
 3. Format the virtual block mapping with an ext4 file system:
   ```bash
   sudo mkfs.ext4 -L SECURE_VOL /dev/mapper/secure_storage
   
   ```
 4. Create a mount point and mount the encrypted volume:
   ```bash
   sudo mkdir -p /mnt/secure_data
   sudo mount /dev/mapper/secure_storage /mnt/secure_data
   
   ```
 5. Verify mount state and writing capabilities:
   ```bash
   df -h /mnt/secure_data
   echo "Top Secret Enterprise Payload" | sudo tee /mnt/secure_data/confidential.txt
   
   ```
### Step 4: LUKS Key Slot Management and Header Backups
 1. **Add a Secondary Recovery Passphrase (Key Slot 1):**
   ```bash
   sudo cryptsetup luksAddKey ${LOOP_DEV}
   
   ```
   * *Enter any existing valid passphrase first, then enter the new secondary recovery passphrase (e.g., RecoveryKey2026!).*
 2. Verify that two key slots are now active:
   ```bash
   sudo cryptsetup luksDump ${LOOP_DEV} | grep -E "Keyslots:|0:|1:"
   
   ```
 3. **Back Up the Binary LUKS Header:**
   The LUKS header contains the keyslots and cipher specifications. If these sectors are damaged, data recovery is impossible.
   ```bash
   sudo mkdir -p /root/luks_backups
   sudo cryptsetup luksHeaderBackup ${LOOP_DEV} --header-backup-file /root/luks_backups/loop0_header.bak
   sudo chmod 400 /root/luks_backups/loop0_header.bak
   
   ```
 4. **Revoke/Kill a Passphrase Key Slot:**
   Demonstrate revoking the secondary recovery key slot (Slot 1):
   ```bash
   sudo cryptsetup luksKillSlot ${LOOP_DEV} 1
   
   ```
 5. Confirm key slot revocation:
   ```bash
   sudo cryptsetup luksDump ${LOOP_DEV} | grep -A 5 "Keyslots:"
   
   ```
### Step 5: Configuring Keyfile Authentication and System Unmounting
 1. Unmount the active encrypted volume and close the device mapping:
   ```bash
   sudo umount /mnt/secure_data
   sudo cryptsetup close secure_storage
   
   ```
 2. Generate a high-entropy 4096-bit random keyfile:
   ```bash
   sudo dd if=/dev/urandom of=/root/luks_volume.key bs=512 count=1
   sudo chmod 400 /root/luks_volume.key
   
   ```
 3. Associate the keyfile with an available key slot in the LUKS header:
   ```bash
   sudo cryptsetup luksAddKey ${LOOP_DEV} /root/luks_volume.key
   
   ```
 4. Test opening the encrypted volume non-interactively using the keyfile:
   ```bash
   sudo cryptsetup open ${LOOP_DEV} secure_storage --key-file /root/luks_volume.key
   sudo mount /dev/mapper/secure_storage /mnt/secure_data
   cat /mnt/secure_data/confidential.txt
   
   ```
 5. Clean up mount state:
   ```bash
   sudo umount /mnt/secure_data
   sudo cryptsetup close secure_storage
   
   ```
### Step 6: File-System Level Encryption Hands-On with eCryptfs
 1. Install eCryptfs utilities (if not present):
   ```bash
   # Debian / Ubuntu
   sudo apt-get install -y ecryptfs-utils
   
   ```
 2. Create source directory targets for directory-level stacked encryption:
   ```bash
   mkdir -p ~/secret_raw ~/secret_enc
   
   ```
 3. Mount ~/secret_raw using eCryptfs with direct CLI option parameters:
   ```bash
   sudo mount -t ecryptfs ~/secret_raw ~/secret_raw \
     -o ecryptfs_cipher=aes,ecryptfs_key_bytes=32,ecryptfs_passthrough=n,ecryptfs_enable_filename_crypto=n,no_sig_cache
   
   ```
   * *When prompted, enter a pass-phrase (e.g., eCryptfsPass2026!). Select defaults for key signature setup.*
 4. Write test files inside the mounted directory:
   ```bash
   echo "File System Encryption Active" > ~/secret_raw/file1.txt
   
   ```
 5. Unmount the eCryptfs instance:
   ```bash
   sudo umount ~/secret_raw
   
   ```
 6. Inspect raw directory contents post-unmount:
   ```bash
   cat ~/secret_raw/file1.txt
   
   ```
   * **Result:** Content is unreadable ciphertext stored directly on the base file system.
### Laboratory Cleanup
Clean up all loopback devices and temporary files created during this lab session:
```bash
sudo losetup -d ${LOOP_DEV} 2>/dev/null || true
rm -f /var/tmp/encrypted_disk.img /root/luks_volume.key /root/luks_backups/loop0_header.bak
rm -rf ~/secret_raw ~/secret_enc /mnt/secure_data

```
## 6.6 Chapter Review & Knowledge Check
### Self-Assessment Questions
#### Question 1
What primary architectural advantage does block-level disk encryption (LUKS) provide over file-system level encryption (eCryptfs)?
 * A) LUKS requires significantly lower CPU and RAM resources for bulk throughput operations.
 * B) LUKS encrypts raw sectors, completely hiding directory structures, file sizes, permissions, and file names.
 * C) LUKS allows different users to mount separate individual subdirectories with distinct passphrases on a shared filesystem.
 * D) LUKS eliminates the need to back up cryptographic metadata headers.
#### Question 2
When changing a user passphrase on a LUKS-encrypted block device using cryptsetup luksChangeKey, what actually happens under the hood?
 * A) The entire physical hard drive payload is re-encrypted using the new passphrase.
 * B) A new random Master Key is generated and used to re-encrypt all existing storage sectors.
 * C) The Master Key remains unchanged; only the targeted LUKS Key Slot holding the encrypted Master Key is updated.
 * D) The LUKS header is wiped and replaced with an unencrypted header block.
#### Question 3
Why is encrypting standard Linux system swap space considered a critical security practice?
 * A) Unencrypted swap partitions increase raw physical disk write latency.
 * B) System RAM contents (including plain-text credentials and decrypted master keys) can be written to unencrypted swap sectors during memory paging or hibernation.
 * C) Cryptsetup cannot open mapped devices unless a swap partition is present.
 * D) Swap space contains the primary Root CA trusted certificate database.
#### Question 4
Which default symmetric cipher mode is recommended for Linux disk encryption to protect against pattern leaks across repeated disk sector blocks?
 * A) aes-ecb-plain
 * B) aes-xts-plain64
 * C) 3des-cbc-raw
 * D) rc4-stream-64
### Answers and Explanations
 1. **Correct Answer: B (LUKS encrypts raw sectors, completely hiding directory structures, file sizes, permissions, and file names)**
   * *Explanation:* Block-level encryption operates below the filesystem. Because the file system index itself resides inside the encrypted blocks, metadata such as file names and directory trees are invisible without unlocking the volume. File-system level encryption like eCryptfs leaves certain metadata exposed.
 2. **Correct Answer: C (The Master Key remains unchanged; only the targeted LUKS Key Slot holding the encrypted Master Key is updated)**
   * *Explanation:* LUKS uses a two-tier key hierarchy. Passphrases unlock individual key slots that decrypt the static Master Key. Updating a passphrase only updates the specified key slot, avoiding time-consuming re-encryption of the whole disk payload.
 3. **Correct Answer: B (System RAM contents—including plain-text credentials and decrypted master keys—can be written to unencrypted swap sectors during memory paging or hibernation)**
   * *Explanation:* Linux pages RAM memory to swap space under high memory pressure or during hibernation. Unencrypted swap space exposes sensitive memory structures to offline extraction.
 4. **Correct Answer: B (aes-xts-plain64)**
   * *Explanation:* AES in XTS mode (aes-xts-plain64) is specifically designed for sector-based block storage to ensure identical plaintext blocks produce distinct ciphertexts across different physical disk offsets.
   
 
