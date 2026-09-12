# Chapter 15: Security Engineering, Hardening, and Access Control
## 15.1 Enterprise Security Layers and Principle of Least Privilege
Hardening a Linux enterprise system requires a layered defense-in-depth security posture. Security control extends beyond standard Discretionary Access Control (DAC) file permissions (rwx) to include Mandatory Access Control (MAC), kernel protection parameters, PAM modules, and automated vulnerability auditing.
```
                   ENTERPRISE LINUX SECURITY LAYERS

   +--------------------------------------------------------------------+
   |                       User / Application Access                    |
   +--------------------------------------------------------------------+
                                     |
                                     v
   +--------------------------------------------------------------------+
   |               Pluggable Authentication Modules (PAM)               |
   |               (/etc/pam.d/ - MFA, Password Quality)                |
   +--------------------------------------------------------------------+
                                     |
                                     v
   +--------------------------------------------------------------------+
   |             Discretionary Access Control (DAC) Permissions          |
   |              (POSIX File Mode, ACLs via setfacl/getfacl)          |
   +--------------------------------------------------------------------+
                                     |
                                     v
   +--------------------------------------------------------------------+
   |             Mandatory Access Control (MAC) Subsystems              |
   |             (SELinux Contexts / AppArmor Security Profiles)        |
   +--------------------------------------------------------------------+
                                     |
                                     v
   +--------------------------------------------------------------------+
   |               Kernel Security & Host Firewall (nftables)           |
   |                (Kernel hardening, sysctl, capabilities)            |
   +--------------------------------------------------------------------+

```
### Security Models Comparison: DAC vs. MAC
| Feature | Discretionary Access Control (DAC) | Mandatory Access Control (MAC) |
|---|---|---|
| **Control Authority** | Resource owner determines access permissions. | System administrator defines central security policy. |
| **Enforcement Primitive** | File modes (chmod), Ownership (chown), POSIX ACLs. | Security contexts/labels (SELinux) or Profiles (AppArmor). |
| **Root Vulnerability** | Compromised root bypasses all restrictions. | root user is constrained by defined system security policies. |
| **Implementation** | Default Linux file permissions (/etc/passwd, mode bits). | SELinux (RHEL/Fedora/Astra) or AppArmor (Debian/Ubuntu). |
## 15.2 Mandatory Access Control (MAC): SELinux & AppArmor
MAC subsystems prevent compromised daemons from escalating privileges or accessing resources outside their declared operational profile.
### SELinux Security Context Structure
SELinux assigns security labels to processes, files, and network ports:
user:role:type:sensitivity (e.g., system_u:object_r:httpd_sys_content_t:s0)
 * **Enforcing Mode:** Blocks unauthorized access and logs policy violations.
 * **Permissive Mode:** Allows unauthorized access but logs policy violations for troubleshooting.
 * **Disabled Mode:** Completely disables the SELinux kernel subsystem.
### SELinux vs. AppArmor Context
| Metric / Aspect | SELinux | AppArmor |
|---|---|---|
| **Primary Mechanism** | Label-based object classification (Types/Domains). | Path-based application profile rules. |
| **Default Distributions** | RHEL, Fedora, Rocky Linux, Astra Linux. | Ubuntu, Debian, SUSE Linux Enterprise. |
| **Policy Complexity** | High (Requires type enforcement rule understanding). | Moderate (Uses explicit path masks and capabilities). |
## 15.3 Pluggable Authentication Modules (PAM)
PAM provides a flexible, modular architecture for system authentication. Configuration files reside in /etc/pam.d/.
### PAM Control Flags
 * **required:** Must succeed for authentication to continue. If it fails, execution continues but overall authentication will ultimately fail.
 * **requisite:** Must succeed. If it fails, authentication fails immediately without executing remaining modules.
 * **sufficient:** If it succeeds, authentication is immediately granted (provided no prior required module failed).
 * **optional:** Success or failure is ignored unless it is the only module in the stack.
## 15.4 Auditd and Compliance Monitoring
The Linux Audit Daemon (auditd) tracks security-relevant events at the kernel level, capturing file accesses, execution calls, and system modifications.
### Common auditctl Rules Syntax
```ini
# Monitor writes/edits to critical system files
-w /etc/passwd -p wa -k identity_changes
-w /etc/shadow -p wa -k identity_changes
-w /etc/sudoers -p wa -k admin_changes

# Monitor execution of specific binaries
-a always,exit -F arch=b64 -S execve -k process_execution

```
## 15.5 Hands-On Lab Framework: Securing Accounts, MAC Enforcement, and Audit Rules
### Lab Prerequisites & Setup
 * **Operating System:** Ubuntu 24.04 LTS or RHEL 9 with sudo privileges.
 * **Required Packages:** auditd, audispd-plugins, apparmor-utils (Ubuntu) or policycoreutils (RHEL).
### Step 1: Enforce Complex Password Policies via PAM
 1. Install the PAM quality management package:
   ```bash
   sudo apt update && sudo apt install -y libpam-pwquality
   
   ```
 2. Configure global password complexity requirements in /etc/security/pwquality.conf:
   ```bash
   sudo tee /etc/security/pwquality.conf << 'EOF'
   minlen = 14
   minclass = 4
   maxrepeat = 2
   gecoscheck = 1
   difok = 5
   EOF
   
   ```
 3. Test password policy validation syntax using pam_pwquality:
   ```bash
   echo "WeakPass1" | pwscore
   # Expect low score output
   echo "Correct-Horse-Battery-Staple-2026!" | pwscore
   # Expect high score (>= 50)
   
   ```
### Step 2: Manage AppArmor / SELinux Application Profiles
#### Option A: Ubuntu / Debian (AppArmor)
 1. Check current status of AppArmor profiles:
   ```bash
   sudo aa-status
   
   ```
 2. Place a service profile (e.g., ping or usr.sbin.tcpdump) into enforce mode:
   ```bash
   sudo aa-enforce /usr/bin/tcpdump
   
   ```
#### Option B: RHEL / Rocky (SELinux)
 1. Verify system SELinux operational status:
   ```bash
   sestatus
   
   ```
 2. Relabel custom web root directory context to standard HTTP server type:
   ```bash
   sudo semanage fcontext -a -t httpd_sys_content_t "/srv/web(/.*)?"
   sudo restorecon -Rv /srv/web
   
   ```
### Step 3: Implement Kernel Audit Rules with auditd
 1. Start and enable the audit daemon:
   ```bash
   sudo systemctl enable --now auditd
   
   ```
 2. Define custom security monitoring rules in /etc/audit/rules.d/hardening.rules:
   ```bash
   sudo tee /etc/audit/rules.d/hardening.rules << 'EOF'
   # Audit changes to sensitive authentication files
   -w /etc/shadow -p wa -k shadow_tamper
   -w /etc/pam.d/ -p wa -k pam_config_change
   
   # Audit privileged executions
   -a always,exit -F path=/usr/bin/sudo -F perm=x -F auid>=1000 -F auid!=4294967295 -k privileged_sudo
   EOF
   
   ```
 3. Load audit rules and verify active policy:
   ```bash
   sudo augenrules --load
   sudo auditctl -l
   
   ```
 4. Trigger an audit log event and query the log:
   ```bash
   sudo touch /etc/shadow
   sudo ausearch -k shadow_tamper --start recent
   
   ```
### Step 4: System Vulnerability Assessment using OpenSCAP
 1. Install OpenSCAP scanner utilities:
   ```bash
   sudo apt install -y libopenscap8 scap-security-guide
   
   ```
 2. Run an automated compliance scan against the official CIS or DISA STIG baseline:
   ```bash
   oscap xccdf eval \
     --profile xccdf_org.ssgproject.content_profile_cis \
     --report /tmp/security_report.html \
     /usr/share/xml/scap/ssg/content/ssg-ubuntu2404-ds.xml || true
   
   ```
 3. Confirm report generation:
   ```bash
   ls -lh /tmp/security_report.html
   
   ```
### Step 5: Clean Up Lab Environment
 1. Revert test audit rules and remove temporary files:
   ```bash
   sudo rm -f /etc/audit/rules.d/hardening.rules
   sudo augenrules --load
   sudo rm -f /tmp/security_report.html
   
   ```
## 15.6 Chapter Review Questions
 1. In Mandatory Access Control (MAC) systems like SELinux, which operational mode enforces policies and actively blocks unauthorized file and port access operations?
   * A) Disabled
   * B) Permissive
   * C) Enforcing
   * D) Audit-Only
 2. Which PAM control flag causes authentication to immediately fail without evaluating remaining modules in the stack upon module failure?
   * A) required
   * B) requisite
   * C) sufficient
   * D) optional
 3. What system tool is used to query and filter events collected by the Linux Audit Daemon (auditd)?
   * A) journalctl
   * B) ausearch
   * C) auditctl -l
   * D) logrotate
 4. What main vulnerability inherent to Discretionary Access Control (DAC) does Mandatory Access Control (MAC) address?
   * A) DAC does not support standard file permissions (rwx).
   * B) DAC allows processes owned by the root user to bypass all standard file mode checks.
   * C) DAC prevents files from being persistently mounted via UUID.
   * D) DAC requires explicit network open port bindings.
## 15.7 Key Terms Glossary
 * **AppArmor:** A path-based Mandatory Access Control (MAC) security framework widely used on Debian and Ubuntu Linux distributions.
 * **auditd:** The Linux Audit Daemon responsible for logging security-relevant kernel events to disk.
 * **Discretionary Access Control (DAC):** Traditional security model where file owners determine permissions for users and groups.
 * **Mandatory Access Control (MAC):** System-wide security enforcement mechanism restricting process capabilities based on central policies regardless of user privileges.
 * **OpenSCAP:** An open-source framework implementing Security Content Automation Protocol (SCAP) for automated system compliance auditing and vulnerability assessment.
 * **PAM (Pluggable Authentication Modules):** A flexible suite of shared libraries that handle user authentication for system services.
 * **SELinux (Security-Enhanced Linux):** A flexible label-based MAC implementation built into the Linux kernel to enforce strict access control rules.
 * **pwquality:** A PAM module used to check the strength and complexity of new passwords against system security criteria.
 
