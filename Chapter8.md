# Chapter 8: Operation & Application Security
## Objective Overview: LPI 022.6 Operations & App Security
| Field | Details |
|---|---|
| **Objective Code** | 022.6 |
| **Weight** | 3 |
| **Description** | Master Linux system auditing (auditd), comprehensive log management (rsyslog, journalctl, log rotation), automated patch management strategies, secure backup paradigms, and essential cloud/virtualization security concepts. |
| **Key Knowledge Areas** | Linux Audit Subsystem (auditd, auditctl, ausearch), System Logging (rsyslog, /etc/rsyslog.conf), Systemd Journal (journalctl), Log Rotation (logrotate), Automated Updates (unattended-upgrades, dnf-automatic), Backup types (Full, Differential, Incremental, 3-2-1 rule), Cloud Security Essentials (IAM, Security Groups, Shared Responsibility Model). |
## 8.1 Linux Audit Subsystem (auditd) Deep Dive
While standard logs record system events, the **Linux Audit Subsystem (auditd)** provides granular, low-level auditing of kernel-level events, tracking *who* did *what* to *which file* or *system call*.
```
+-----------------------------------------------------------------------------------+
|                            AUDITD ARCHITECTURE                                    |
+-----------------------------------------------------------------------------------+

     [ Userspace ] <--- (Configuration & Query Tools) ---> [ auditctl / ausearch ]
            |                                                      ^
            v                                                      |
     [ auditd Daemon ] <-------------------+                       |
            |                             |                       |
            v                             |                       |
     [ Audit Logs ] (/var/log/audit/audit.log)                   |
                                          |                       |
------------------------------------------|-----------------------|-----------------
     [ Kernel ]                           |                       |
                                          |                       |
     [ System Call Interface ] <----------+                       |
            |                                                      |
            v                                                      |
     [ Netlink Audit Multicast ] ----------------------------------+

```
### 1. Core Audit Subsystem Components
 * **auditd:** The userspace daemon that receives audit events from the kernel and writes them to /var/log/audit/audit.log.
 * **auditctl:** Utility used to configure audit rules dynamically (controlling system call, file, or executive auditing).
 * **ausearch:** Powerful query tool used to search through recorded audit logs based on events, users, files, or specific timeframes.
 * **/etc/audit/audit.rules:** Persistent configuration file holding the audit rules loaded at system boot.
## 8.2 Logging Architectures: rsyslog and journalctl
Linux systems employ hybrid logging models, utilizing standard syslog daemons alongside the modern systemd journal.
```
+------------------------------------+------------------------------------+
| rsyslog                            | systemd-journald                   |
+------------------------------------+------------------------------------+
| Traditional syslog protocol (RFC 5424)| Binary, structured logging format |
| Configuration via `/etc/rsyslog.conf`| Configuration via `/etc/systemd/journald.conf`|
| Writes plain text logs to `/var/log/`| Stores logs in RAM (temporary) or |
|                                    | `/var/log/journal/` (persistent)    |
| Powerful filtering & remote forwarding| Fast searching, integral to systemd |
| Tooling: `logger`, text viewers      | Tooling: `journalctl`               |
+------------------------------------+------------------------------------+
| Combined Model: `systemd-journald` forwards logs to `rsyslog`.        |
+------------------------------------+------------------------------------+

```
### Log Rotation (logrotate)
Standard plain-text logs grow indefinitely if unmanaged. **logrotate** is a utility (run periodically via cron/timers) that rotates, compresses, and purges old log files based on age or size configuration in /etc/logrotate.conf and /etc/logrotate.d/.
## 8.3 Operation & Patch Management Security Best Practices
Securing a running system requires robust operational procedures, including automated maintenance and data preservation.
```
+-------------------------------------------------------------------------+
|                  OPERATIONAL SECURITY BEST PRACTICES                    |
+-------------------------------------------------------------------------+
|  1. Automated Patching:                                                 |
|     Configure `unattended-upgrades` (Debian/Ubuntu) or `dnf-automatic`   |
|     (RHEL/Fedora) to automatically install critical security updates.     |
|                                                                         |
|  2. Secure Backup Architectures:                                        |
|     Follow the 3-2-1 Rule: 3 copies of data, 2 different media types,   |
|     1 copy offsite. Backups MUST be encrypted at rest and in transit.     |
|                                                                         |
|  3. Cloud Security Governance:                                          |
|     Understand the Shared Responsibility Model: Cloud Provider secures   |
|     the infrastructure; Customer secures OS, Apps, and Data.             |
|                                                                         |
|  4. Principle of Least Privilege:                                       |
|     Utilize IAM roles, restricted service accounts, and strict Cloud    |
|     Security Group rules to deny all traffic by default.               |
+-------------------------------------------------------------------------+

```
## 8.4 Practical Scenario: Implementation Audit Logging and Secure Data Preservation
### Scenario Context
You are a Senior Security Systems Administrator for **Cybergate Services Private Limited**. A critical internal server houses sensitive R&D documentation. You must implement advanced auditing to track any modifications to files within the research directory, configure a remote central logging pipeline using rsyslog, automate security patching, and execute an encrypted incremental backup.
```
[ Research Server Node ]
  ├── Active File Access Auditing (auditd)
  ├── Remote Log Forwarding (rsyslog)
  ├── Automated Security Patching
  └── Encrypted 3-2-1 Backups

```
## 8.5 Hands-On Laboratory: Audit Auditing, Journal Management, and Backups
In this lab, you will configure file system auditing using auditd, manage binary logs with journalctl, and create an encrypted, multi-volume backup.
### Prerequisites
 * A Linux host (Ubuntu/Debian or RHEL family) with root or sudo privileges.
 * Install required packages:
   ```bash
   # Debian / Ubuntu
   sudo apt-get update && sudo apt-get install -y auditd rsyslog systemd-journal-remote tar gnupg
   
   ```
### Step 1: Auditing File Access and Process Execution with auditd
 1. Start the audit daemon and check its status:
   ```bash
   sudo systemctl enable --now auditd
   sudo systemctl status auditd
   
   ```
 2. **Add a File Integrity Audit Rule:** Create a rule to monitor all read (r), write (w), execute (x), and attribute (a) changes to /etc/passwd. Attach a searchable key identifier passwd_mods.
   ```bash
   sudo auditctl -w /etc/passwd -p rwa -k passwd_mods
   
   ```
   * *Flag Breakdown:* -w: Watch path; -p: Permissions to monitor; -k: Custom searchable key.
 3. Simulate a non-privileged user trying to read /etc/passwd:
   ```bash
   cat /etc/passwd
   
   ```
 4. **Add a Process Execution Audit Rule:** Create a rule to audit the execution of the chmod system call (used for altering permissions) by any non-root user (UID >= 1000), tagging it with the key perm_mods.
   ```bash
   sudo auditctl -a always,exit -F arch=b64 -S chmod -F "uid>=1000" -k perm_mods
   
   ```
   * *Flag Breakdown:* -a: Append rule to list; -F arch: Architecture filtering; -S: System call filtering.
 5. Simulate a user execution of chmod:
   ```bash
   touch ~/test_file
   chmod 700 ~/test_file
   
   ```
 6. **Search Audit Logs using ausearch:**
   * Query all events tagged with the key passwd_mods:
     ```bash
     sudo ausearch -k passwd_mods
     
     ```
   * Query audit events recorded within the last 5 minutes:
     ```bash
     sudo ausearch -ts recent
     
     ```
 7. List all currently active dynamic audit rules:
   ```bash
   sudo auditctl -l
   
   ```
### Step 2: Advanced Journal Management with journalctl
 1. Display the tail of the system journal in real-time (similar to tail -f):
   ```bash
   sudo journalctl -f
   
   ```
 2. Display journal entries recorded between specific timestamps:
   ```bash
   sudo journalctl --since "2026-10-10 12:00:00" --until "2026-10-10 13:00:00"
   
   ```
 3. View all log entries generated exclusively by a specific systemd unit (e.g., ssh):
   ```bash
   sudo journalctl -u ssh
   
   ```
 4. View logs generated by a specific system process ID (PID) or executable binary path:
   ```bash
   sudo journalctl /usr/sbin/sshd
   
   ```
 5. Filter the journal by severe priority levels (Crit=2, Alert=1, Emerg=0):
   ```bash
   sudo journalctl -p 2
   
   ```
### Step 3: Configuring Remote Syslog Logging (rsyslog)
Perform this step within a simulated client/server setup if possible, or simulate local forwarding for demonstration.
 1. **Configure rsyslog to receive remote logs (Server Role):**
   * Edit /etc/rsyslog.conf.
   * Uncomment the following lines to enable UDP syslog reception on port 514:
     ```text
     module(load="imudp")
     input(type="imudp" port="514")
     
     ```
 2. **Configure rsyslog to forward logs to a remote server (Client Role):**
   * Forward all critical logs (*.crit) via UDP to a central server (e.g., IP 192.168.100.250). Add this line to /etc/rsyslog.conf:
     ```text
     *.crit @192.168.100.250:514
     
     ```
     * *Note:* Use a single @ for UDP forwarding, double @@ for TCP forwarding.
 3. Restart rsyslog to apply changes:
   ```bash
   sudo systemctl restart rsyslog
   
   ```
 4. Generate a test critical syslog event using the logger utility:
   ```bash
   logger -p crit "CRITICAL SYSTEM ALERT: Cybergate R&D Database connection failure."
   
   ```
### Step 4: Executing a Secure, Encrypted, Split Backup Pipeline
Simulate securing a sensitive dataset and preparing it for offsite storage.
 1. Create a simulated sensitive dataset:
   ```bash
   sudo mkdir -p /opt/cybergate/research
   echo "Top Secret Research Payload V1" | sudo tee /opt/cybergate/research/v1.txt
   echo "Top Secret Research Payload V2" | sudo tee /opt/cybergate/research/v2.txt
   
   ```
 2. Create a backup target directory:
   ```bash
   mkdir -p ~/backups
   
   ```
 3. **Secure Backup Pipeline:** Execute a single-line pipeline that archives the research data, compresses it, splits it into 10MB multi-volume volumes, and encrypts the final output using a symmetric passphrase via GnuPG.
   ```bash
   sudo tar -czvf - /opt/cybergate/research | \
     gpg --symmetric --cipher-algo AES256 -c | \
     split -b 10M - ~/backups/research_backup_$(date +%F).tar.gz.gpg.
   
   ```
   * *Enter a strong backup passphrase when prompted (e.g., BackupCryptoPass2026!).*
 4. Inspect the resulting multi-volume encrypted backup files:
   ```bash
   ls -lh ~/backups/
   
   ```
 5. Clean up the source sensitive data (to simulate disaster scenario):
   ```bash
   sudo rm -rf /opt/cybergate/research
   
   ```
### Step 5: Disaster Recovery – Decrypting and Restoring Multi-Volume Backups
 1. Verify that the current directory does not contain the research data:
   ```bash
   ls -l /opt/cybergate/research 2>/dev/null || echo "Data is gone."
   
   ```
 2. **Restore Pipeline:** Concatenate the multi-volume parts back together, decrypt the symmetric payload, decompress, and extract the original data structure.
   ```bash
   cat ~/backups/research_backup_*.tar.gz.gpg.* | \
     gpg --decrypt | \
     sudo tar -xzvf - -C /
   
   ```
   * *Enter the passphrase defined in Step 4 when prompted.*
 3. Verify data restoration:
   ```bash
   cat /opt/cybergate/research/v2.txt
   
   ```
### Laboratory Cleanup
Clean up output trace files and backup artifacts:
```bash
sudo rm -rf /opt/cybergate/research ~/backups

```
## 8.6 Chapter Review & Knowledge Check
### Self-Assessment Questions
#### Question 1
Which specific Linux Audit Subsystem utility is used to define dynamic rules controlling system calls, file integrity watches, and privilege escalation auditing at the kernel level?
 * A) ausearch
 * B) auditctl
 * C) ureport
 * D) logrotate
#### Question 2
You suspect a security compromise occurred on an application node between 14:00 and 15:00 yesterday. Which journalctl command best isolates binary system logs recorded within that specific timeframe?
 * A) journalctl --since "14:00 yesterday" --until "15:00 yesterday"
 * B) journalctl --priority 3 --time "14:00-15:00"
 * C) journalctl --unit appserver --since -1h
 * D) journalctl --pid appserver --until -1d
#### Question 3
Following the 3-2-1 backup security standard, which component is universally mandatory for any backup media containing sensitive organizational data?
 * A) Compression via gzip.
 * B) Multi-volume splitting using split.
 * C) Encryption at rest.
 * D) Periodic restoration verification.
#### Question 4
Under the Cloud Security Shared Responsibility Model, which component is typically listed as the responsibility of the cloud *customer*, rather than the cloud provider?
 * A) Physical security of the data center hardware.
 * B) Hypervisor patching and virtualization security.
 * C) Configuration of network security groups and IAM permissions.
 * D) Physical storage device disposal.
### Answers and Explanations
 1. **Correct Answer: B (auditctl)**
   * *Explanation:* auditctl is the CLI tool for interacting with the audit kernel subsystem to load, append, or list audit rules.
 2. **Correct Answer: A (journalctl --since "14:00 yesterday" --until "15:00 yesterday")**
   * *Explanation:* journalctl supports powerful human-readable time strings like "yesterday" alongside explicit timestamps to filter binary log entries.
 3. **Correct Answer: C (Encryption at rest)**
   * *Explanation:* Regardless of backup type or media (3-2-1 rule), sensitive data must always be encrypted at rest on the backup media to protect against theft or exposure.
 4. **Correct Answer: C (Configuration of network security groups and IAM permissions)**
   * *Explanation:* Cloud providers secure the "cloud" infrastructure (physical hardware, hypervisors), while customers secure their data, identities, operating systems, and network configurations (security groups, IAM).
   
