# Chapter 1: Core Security Frameworks and Threat Actors
## Objective Overview: LPI 021.1 Security Concepts
| Field | Details |
|---|---|
| **Objective Code** | 021.1 |
| **Weight** | 1 |
| **Description** | Understand the fundamental goals of IT security, common actors, roles, attack motivations, and the technical challenges surrounding attribution. |
| **Key Knowledge Areas** | CIA Triad, Non-Repudiation, Threat Actor Profiles, Industrial Espionage, Service Interruption, Ransomware, Technical Attribution. |
## 1.1 Fundamentals of the CIA Triad and Non-Repudiation
Information security frameworks prioritize three core attributes, collectively known as the **CIA Triad**:
```
                  +-----------------------+
                  |  CONFIDENTIALITY      |
                  |  (Encryption / ACLs)  |
                  +-----------+-----------+
                              |
                              |
            +-----------------+-----------------+
            |                                   |
+-----------v-----------+           +-----------v-----------+
|      INTEGRITY        |           |     AVAILABILITY      |
| (Hashing / Digital    |           | (Redundancy / DoS     |
|   Signatures)         |           |     Mitigation)       |
+-----------------------+           +-----------------------+

```
### 1. Confidentiality
Ensures that sensitive data is accessible only to authorized entities and remains hidden from unauthorized eyes.
 * **Control Mechanisms:** Symmetric/asymmetric encryption, File System Access Control Lists (ACLs), POSIX permissions (chmod, chown), and network transport encryption (TLS).
### 2. Integrity
Guarantees that data and systems remain authentic, accurate, and safe from unauthorized modification or deletion.
 * **Control Mechanisms:** Cryptographic checksums (sha256sum), digital signatures (gpg), file system integrity monitors (tripwire, AIDE), and database constraints.
### 3. Availability
Ensures systems, network services, and critical data are operational and accessible when authorized users require them.
 * **Control Mechanisms:** High Availability (HA) clustering, Load balancing, Data backups, Disaster Recovery (DR) protocols, and Denial of Service (DoS) mitigation.
### 4. Non-Repudiation
Non-repudiation ensures that an actor cannot deny the authenticity of their signature on a document or the sending of a message that they originated.
 * **Control Mechanisms:** Asymmetric cryptography (digital signatures) and cryptographically bound, immutable system logging (auditd remote logging, signed syslog events).
## 1.2 Profiling Threat Actors and Attack Motivations
Modern system defenders categorize security risks based on the capabilities, access level, and motives of potential attackers.
```
+------------------+----------------------------------+------------------------------------+
| Threat Actor     | Motivations                      | Capabilities                       |
+------------------+----------------------------------+------------------------------------+
| Script Kiddies   | Notoriety, curiosity, vandalism  | Pre-written scripts, public exploits|
| Hacktivists      | Political / Ideological goals    | Defacement, DoS, leak campaigns    |
| Cybercriminals   | Financial gain, extortion        | Ransomware, phishing, botnets      |
| Nation-States    | Espionage, disruption, sabotage | Zero-days, custom malware, APTs    |
| Insiders         | Revenge, financial gain          | Valid credentials, internal access |
+------------------+----------------------------------+------------------------------------+

```
### Threat Actor Taxonomy
 1. **Script Kiddies (Unskilled Actors):** Individuals who execute existing, publicly available exploit kits without understanding the underlying mechanisms or code.
 2. **Black Hat Hackers:** Threat actors who violate system integrity for malicious intent, financial gain, or personal satisfaction.
 3. **White Hat Hackers (Ethical Hackers):** Security professionals who apply offensive skills to find and remediate security vulnerabilities within authorized scopes.
 4. **Grey Hat Hackers:** Individuals who may break laws or ethical standards during vulnerability research, but without the explicit malicious intent typical of a Black Hat.
 5. **Advanced Persistent Threats (APTs):** Highly funded, state-sponsored entities or sophisticated criminal syndicates that conduct sustained, stealthy, long-term cyber intrusion operations.
### Common Motives Behind Attacks
 * **Data Access and Theft:** Stealing intellectual property, customer databases, or trade secrets (**Industrial Espionage**).
 * **Data Manipulation & Destruction:** Altering system databases or corrupting operational data to undermine organizational trust.
 * **Extortion & Ransom:** Encrypting critical operational files and demanding payment for decryption keys (**Ransomware**).
 * **Service Disruption:** Overwhelming infrastructure resources to take critical web services offline (**Denial of Service**).
## 1.3 The Technical Challenge of Attribution
**Attribution** refers to the process of definitively identifying the individual, group, or nation-state responsible for a cyber intrusion or service disruption.
### Why Attribution Is Difficult
 1. **Proxy Networks and Anonymizers:** Attackers route malicious traffic through compromised intermediaries, Virtual Private Networks (VPNs), or Onion Routing systems (Tor).
 2. **False Flag Tactics:** Sophisticated attackers intentionally incorporate code styles, language strings, or tools associated with *other* known threat groups to mislead forensic investigators.
 3. **Compromised Staging Infrastructure:** Attackers rarely use their own infrastructure; they hijack vulnerable third-party web servers or cloud instances to launch exploits.
 4. **Shared Exploit Tooling:** The widespread availability of open-source penetration testing tools (e.g., Metasploit, Nmap) makes distinguishing between different threat groups using similar tools challenging.
## 1.4 Real-World Scenario: Enterprise Threat Modeling
### Scenario Context
You are an IT Security Specialist for **Apex Logistics Inc.**. The primary customer portal is an internal web gateway running on a Linux infrastructure. During a vulnerability assessment, an unauthenticated **Remote Code Execution (RCE)** vulnerability is discovered in the public-facing HTTP interface.
```
[Attacker] ---> ( Internet ) ---> [Apex Gateway: Port 80/443] ---> [Internal DB Server]
                                          |
                              (Unauthenticated RCE)

```
### Threat Analysis & Impact Matrix
 1. **CIA Impact Analysis:**
   * **Confidentiality:** High risk. An unauthenticated attacker executing remote code can dump system environment variables, extract database credentials, and gain access to client shipment records.
   * **Integrity:** High risk. The attacker can modify system configurations, inject webshells, or alter database audit records.
   * **Availability:** Critical risk. An attacker can deploy ransomware encrypting local block storage or execute system shutdown routines (poweroff, rm -rf /).
 2. **Threat Actor Profile:**
   * **Primary Threat:** Automated botnets and Cybercriminals scanning public IPv4 space for vulnerable endpoints.
   * **Motive:** Financial extortion via automated deployment of ransomware or silent recruitment of the host into a Distributed Denial of Service (DDoS) botnet.
 3. **Immediate Mitigation Strategy:**
   * Isolate the web server at the network firewall layer (block external access to ports 80/443).
   * Review local process tables (ps aux, ss -tulpn) and system logs (/var/log/syslog, journalctl) for signs of active exploitation.
   * Apply software updates or deploy an emergency virtual patch at the Web Application Firewall (WAF) layer.
## 1.5 Hands-On Laboratory: Inspecting System Integrity and Logging
In this hands-on lab, you will use Linux CLI utilities to analyze service availability, inspect running processes, verify file integrity, and check system logs.
### Prerequisites
A Linux system (Ubuntu/Debian, RHEL, or Fedora) with root or sudo administrative access.
### Step 1: Auditing Running Services & Network Availability
 1. Open a terminal and check all active network listeners on your system:
   ```bash
   ss -tulpn
   
   ```
   * *Output Explanation:*
     * -t: Display TCP sockets.
     * -u: Display UDP sockets.
     * -l: Display listening sockets.
     * -p: Show the process using the socket.
     * -n: Show numerical addresses instead of resolving service names.
 2. Verify if a specific process (e.g., sshd or nginx) is running using ps and grep:
   ```bash
   ps aux | grep -E "(sshd|nginx|apache2)"
   
   ```
### Step 2: Verifying File Integrity using SHA-256 Hashes
To ensure the integrity of critical configuration files (such as /etc/passwd), administrators construct baseline hashes to detect unauthorized modifications.
 1. Generate a SHA-256 hash for /etc/passwd and save it to a secure reference file:
   ```bash
   sha256sum /etc/passwd | sudo tee /root/passwd.sha256
   
   ```
 2. Inspect the generated baseline file:
   ```bash
   cat /root/passwd.sha256
   
   ```
 3. Verify file integrity against the reference hash:
   ```bash
   sha256sum -c /root/passwd.sha256
   
   ```
   * **Expected Output:** /etc/passwd: OK
 4. *Simulate an Integrity Test:* Append a blank space/comment to a temporary file copy and verify how checksum validation flags tampering:
   ```bash
   cp /etc/passwd /tmp/passwd_test
   sha256sum /tmp/passwd_test > /tmp/passwd_test.sha256
   
   # Modify the test file
   echo "# Tampered comment" >> /tmp/passwd_test
   
   # Run verification
   sha256sum -c /tmp/passwd_test.sha256
   
   ```
   * **Expected Output:** /tmp/passwd_test: FAILED
   * **Security Takeaway:** Hashing ensures file **Integrity** by flagging even single-bit modifications.
### Step 3: Auditing Log File Integrity and Authentication Attempts
Linux logs record system events, providing auditing mechanisms that support **Non-Repudiation** and forensic investigations.
 1. Inspect real-time system logs using journalctl:
   ```bash
   sudo journalctl -n 20 --no-pager
   
   ```
 2. Filter for failed login attempts to inspect potential authentication attacks:
   * **On Debian/Ubuntu systems:**
     ```bash
     sudo grep "Failed password" /var/log/auth.log
     
     ```
   * **On RHEL/CentOS systems:**
     ```bash
     sudo grep "Failed password" /var/log/secure
     
     ```
   * Alternatively, use systemd journal:
     ```bash
     sudo journalctl -u ssh | grep "Failed"
     
     ```
 3. Display the history of successful system logins to verify user accountability:
   ```bash
   last -n 10
   
   ```
## 1.6 Chapter Review & Knowledge Check
### Self-Assessment Questions
#### Question 1
An attacker modifies an internal payroll database to increase their salary. Which leg of the CIA triad has been directly violated?
 * A) Confidentiality
 * B) Integrity
 * C) Availability
 * D) Non-repudiation
#### Question 2
Which type of threat actor is characterized by low technical skills, relying almost entirely on automated, publicly available exploit scripts?
 * A) Advanced Persistent Threat (APT)
 * B) Black Hat
 * C) Script Kiddie
 * D) Hacktivist
#### Question 3
Why is attribution difficult during a post-incident forensic investigation?
 * A) System logs are always deleted automatically after an exploit.
 * B) Threat actors frequently route attacks through compromised proxy hosts and use common tooling.
 * C) Symmetric encryption prevents identifying network source addresses.
 * D) Hashing algorithms obscure source IP addresses in TCP packet headers.
#### Question 4
Which mechanism provides mathematical proof that a configuration file has not been altered since its creation?
 * A) Asymmetric Encryption
 * B) Cryptographic Hash Checksum
 * C) Access Control List (ACL)
 * D) Stateful Firewall Filtering
### Answers and Explanations
 1. **Correct Answer: B (Integrity)**
   * *Explanation:* Integrity guarantees that data remains accurate and protected against unauthorized modification or deletion. Modifying records in a database violates integrity.
 2. **Correct Answer: C (Script Kiddie)**
   * *Explanation:* Script Kiddies lack deep technical expertise and rely on pre-packaged tools and scripts written by others.
 3. **Correct Answer: B (Threat actors frequently route attacks through compromised proxy hosts and use common tooling)**
   * *Explanation:* Attackers obfuscate their origins using proxies, VPNs, Tor, and shared exploit kits, making true source attribution challenging.
 4. **Correct Answer: B (Cryptographic Hash Checksum)**
   * *Explanation:* SHA-256 or SHA-512 cryptographic hashes generate a unique digest of file contents. If a single byte is changed, the calculated hash will differ, revealing modification.
   
