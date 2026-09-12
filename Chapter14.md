# Chapter 14: System Monitoring, Logging, and Performance Tuning
## 14.1 System Logging Architecture and Journald Integration
Linux systems track system health, security events, and service operations using system logging services. Modern Linux distributions utilize a hybrid approach combining systemd-journald for low-level, binary structured event capture and rsyslogd for text-based logging and remote log forwarding.
```
                  ENTERPRISE LINUX LOGGING ARCHITECTURE

   +--------------------------------------------------------------------+
   |               Kernel, Daemons, and User Applications               |
   +--------------------------------------------------------------------+
                                     |
                                     v
   +--------------------------------------------------------------------+
   |                          systemd-journald                          |
   |           (Captures early boot, stdout/stderr, syslog streams)      |
   |              Binary Storage: /run/log/journal/ (volatile)          |
   |                             /var/log/journal/ (persistent)         |
   +--------------------------------------------------------------------+
                                     |
                                     v
   +--------------------------------------------------------------------+
   |                               rsyslogd                             |
   |          (Processes rules in /etc/rsyslog.conf & /etc/rsyslog.d/)   |
   +--------------------------------------------------------------------+
                                     |
        +----------------------------+----------------------------+
        |                                                         |
        v                                                         v
+-------------------------------+                         +--------------------+
|       Local Text Logs         |                         |  Remote Log Server |
| /var/log/syslog or /var/log/messages                    | (SIEM / Syslog)    |
| /var/log/auth.log or /var/log/secure                    +--------------------+
+-------------------------------+

```
### Syslog Log Severities and Facilities
Syslog categorizes messages by **Facility** (source of the log) and **Severity** (importance level).
| Severity Level | Name | Description |
|---|---|---|
| **0** | emerg (Emergency) | System is unusable (e.g., hardware panic). |
| **1** | alert | Action must be taken immediately. |
| **2** | crit (Critical) | Critical conditions (e.g., disk drive failure). |
| **3** | err (Error) | Error conditions (e.g., service startup failure). |
| **4** | warning | Warning conditions. |
| **5** | notice | Normal but significant condition. |
| **6** | info | Informational messages. |
| **7** | debug | Debug-level messages. |
## 14.2 Performance Metrics: Load Averages, CPU, and Memory
Diagnosing performance bottlenecks requires interpreting real-time resource usage metrics across CPU, RAM, Swap, and Disk I/O.
### Understanding Load Averages
Load average represents the average number of threads in a runnable state (using CPU) or uninterruptible sleep state (waiting for Disk/Network I/O) over 1, 5, and 15-minute intervals.
 * **Single-core System:** Load Average of 1.0 means 100% CPU utilization.
 * **Quad-core System:** Load Average of 4.0 means 100% aggregate CPU utilization.
 * **Load > Core Count:** Indicates process queuing and potential resource bottlenecks.
### Essential Performance Tooling
| Tool | Focus | Use Case |
|---|---|---|
| top / htop | CPU, RAM, Processes | Interactive real-time resource and task management. |
| vmstat | Virtual Memory, CPU, Processes | Periodic snapshots of system memory, swap, and CPU wait times. |
| iostat | Disk I/O | I/O load per block device (sysstat package). |
| pidstat | Process-level metrics | Per-process CPU, memory, and task switching tracking. |
| uptime | System Uptime & Load | Quick check of system uptime and 1/5/15-minute load averages. |
## 14.3 Kernel Tuning via /proc and /etc/sysctl.conf
The Linux kernel exposes runtime operational parameters under the /proc/sys/ virtual file system. System administrators adjust these parameters on the fly or make them persistent using sysctl.
### Critical Tuning Parameters
```ini
# /etc/sysctl.d/99-performance.conf

# Balance VM swappiness (0 = avoid swap, 100 = aggressive swap)
vm.swappiness = 10

# Increase maximum file descriptors open system-wide
fs.file-max = 2097152

# Increase network connection backlog limit
net.core.somaxconn = 4096

# Enable TCP SYN Cookies (protects against SYN flood attacks)
net.ipv4.tcp_syncookies = 1

# Enable IP Packet Forwarding
net.ipv4.ip_forward = 1

```
## 14.4 Log Rotation with Logrotate
To prevent text log files from consuming all available disk space, logrotate systematically rotates, compresses, and purges log files based on file size or time intervals.
### Sample Configuration (/etc/logrotate.d/custom-app)
```text
/var/log/custom-app/*.log {
    daily
    rotate 7
    missingok
    compress
    delaycompress
    notifempty
    create 0640 www-data customapp
    sharedscripts
    postrotate
        /usr/bin/systemctl reload custom-app.service > /dev/null 2>&1 || true
    endscript
}

```
## 14.5 Hands-On Lab Framework: Log Analysis, Metric Tracking, and Kernel Tuning
### Lab Prerequisites & Setup
 * **Operating System:** Ubuntu 24.04 LTS or RHEL 9 system with sudo access.
 * **Required Packages:** sysstat, htop, stress-ng, logrotate.
### Step 1: Advanced Log Querying with journalctl
 1. View all logs generated since the current system boot:
   ```bash
   sudo journalctl -b
   
   ```
 2. Filter logs for a specific service (e.g., sshd or systemd-resolved) within a time window:
   ```bash
   sudo journalctl -u systemd-resolved --since "1 hour ago"
   
   ```
 3. Display priority-specific logs (e.g., Error level or worse):
   ```bash
   sudo journalctl -p err..emerg -n 20
   
   ```
 4. Inspect logs in real-time follow mode (similar to tail -f):
   ```bash
   sudo journalctl -f -u ssh
   
   ```
### Step 2: Resource Bottleneck Analysis and Load Testing
 1. Install system monitoring and load testing tools:
   ```bash
   sudo apt update && sudo apt install -y sysstat htop stress-ng
   
   ```
 2. Capture baseline memory and CPU statistics using vmstat (1-second intervals for 5 iterations):
   ```bash
   vmstat 1 5
   
   ```
 3. Open a secondary terminal and execute a controlled CPU stress load:
   ```bash
   stress-ng --cpu 2 --timeout 30s
   
   ```
 4. Observe real-time CPU states in the main terminal:
   ```bash
   mpstat -P ALL 1 5
   
   ```
 5. Monitor per-device disk I/O metrics:
   ```bash
   iostat -xz 1 3
   
   ```
### Step 3: Configure Custom Syslog Rules and Rotation
 1. Create a custom rsyslog rule to isolate authentication events:
   ```bash
   sudo tee /etc/rsyslog.d/45-custom-auth.conf << 'EOF'
   auth,authpriv.* /var/log/custom_auth.log
   EOF
   
   ```
 2. Restart rsyslog service and verify file creation:
   ```bash
   sudo systemctl restart rsyslog
   logger -p auth.notice "Test custom authentication message"
   sudo cat /var/log/custom_auth.log
   
   ```
 3. Define a dedicated logrotate configuration for the custom log:
   ```bash
   sudo tee /etc/logrotate.d/custom-auth << 'EOF'
   /var/log/custom_auth.log {
       size 100k
       rotate 3
       compress
       missingok
       notifempty
       create 0600 root root
   }
   EOF
   
   ```
 4. Dry-run logrotate to verify policy syntax:
   ```bash
   sudo logrotate -d /etc/logrotate.d/custom-auth
   
   ```
### Step 4: Runtime Kernel Optimization with Sysctl
 1. Check current runtime swappiness value:
   ```bash
   cat /proc/sys/vm/swappiness
   
   ```
 2. Change swappiness on-the-fly without rebooting:
   ```bash
   sudo sysctl -w vm.swappiness=15
   
   ```
 3. Make the configuration persistent across reboots:
   ```bash
   sudo tee /etc/sysctl.d/90-sysadmin-tuning.conf << 'EOF'
   # Custom System Tuning
   vm.swappiness = 15
   net.core.somaxconn = 2048
   EOF
   
   ```
 4. Apply settings from sysctl configuration files:
   ```bash
   sudo sysctl --system
   
   ```
### Step 5: Clean Up Lab Environment
 1. Remove created lab configuration files and custom logs:
   ```bash
   sudo rm -f /etc/rsyslog.d/45-custom-auth.conf
   sudo rm -f /etc/logrotate.d/custom-auth
   sudo rm -f /etc/sysctl.d/90-sysadmin-tuning.conf
   sudo rm -f /var/log/custom_auth.log
   sudo systemctl restart rsyslog
   
   ```
## 14.6 Chapter Review Questions
 1. Which journalctl command option limits outputs to log entries recorded during the current system boot session?
   * A) -f
   * B) -u
   * C) -b
   * D) -p
 2. What numerical syslog severity level corresponds to a crit (Critical) condition message?
   * A) 0
   * B) 1
   * C) 2
   * D) 5
 3. On a system with 4 CPU cores, what does a 1-minute load average of 4.0 indicate?
   * A) The system is completely idle.
   * B) The CPUs are utilized at 100% capacity without queuing tasks.
   * C) The system is undergoing severe memory paging.
   * D) The kernel has panicked.
 4. Which command dynamically reloads sysctl parameters from configuration files in /etc/sysctl.d/?
   * A) sysctl -w
   * B) sysctl -a
   * C) sysctl --system
   * D) systemctl reload kernel
## 14.7 Key Terms Glossary
 * **iostat:** A system monitoring tool used to collect and display storage device input/output statistics.
 * **journald:** The systemd logging daemon that collects and manages binary, structured log records.
 * **Logrotate:** A Linux utility designed to simplify the administration of system log files through automatic rotation, compression, and deletion.
 * **mpstat:** A tool that reports individual or aggregated processor usage statistics.
 * **rsyslogd:** An enhanced syslog daemon providing modular inputs, outputs, and message filtering options.
 * **Syslog:** An industry-standard protocol used to send and receive system logging messages.
 * **sysctl:** A command-line utility used to examine and modify kernel parameters at runtime.
 * **vmstat:** Virtual memory statistics tool reporting information about processes, memory, paging, block I/O, traps, and CPU activity.
 
