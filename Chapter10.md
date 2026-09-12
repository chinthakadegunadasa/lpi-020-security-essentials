# Chapter 10: Security Practice Tests & Exam Simulation
## Objective Overview: LPI 020 Security Essentials Final Assessment
| Field | Details |
|---|---|
| **Objective Domain** | Comprehensive Coverage across Topics 1–6 (LPI 020-100 Security Essentials) |
| **Weight Total** | 15 (Full Exam Equivalent) |
| **Description** | Validate knowledge across foundational security goals, risk assessment frameworks, cryptographic primitives, host and storage security, network protection, operations, identity management, and compliance standards. |
| **Key Knowledge Areas** | CIA Triad, Threat Actors, Vulnerability Scanning (nmap, CVEs), Cryptography & PKI (openssl, gpg, LUKS), Netfilter/Firewall Rules (iptables, nftables), System Auditing (auditd), Logging (rsyslog, journalctl), PAM & MFA, and GDPR / Privacy Frameworks. |
## 10.1 Exam Strategy & Blueprint Summary
The **LPI Security Essentials (Exam 020)** tests fundamental security knowledge and practical system administration tasks across six primary domains.
```
+-----------------------------------------------------------------------------------+
|                        LPI 020 EXAM DOMAIN WEIGHT DISTRIBUTION                    |
+-----------------------------------------------------------------------------------+

  [ 1. Security Goals & Roles ]     ■■ (Weight 1)
  [ 2. Risk Assessment & Mgmt ]     ■■■■ (Weight 2)
  [ 3. Encryption & PKI ]           ■■■■■■ (Weight 3)
  [ 4. Device & Storage Security ]  ■■■■ (Weight 2)
  [ 5. Network & Service Security ] ■■■■■■■■ (Weight 4)
  [ 6. Identity & Privacy ]         ■■■■■■ (Weight 3)

```
## 10.2 Comprehensive Practice Exam (40 Questions)
### Question 1 (Domain 1: Security Goals)
An attacker intercepts confidential files sent over an unencrypted Wi-Fi connection. Which leg of the CIA Triad has been directly breached?
 * A) Availability
 * B) Confidentiality
 * C) Integrity
 * D) Non-repudiation
### Question 2 (Domain 1: Threat Actors)
Which category of threat actor is characterized by highly sophisticated capabilities, sustained financial backing, and long-term espionage objectives against critical infrastructure?
 * A) Script Kiddie
 * B) Hacktivist
 * C) Advanced Persistent Threat (APT)
 * D) Insider Threat
### Question 3 (Domain 2: Risk Assessment)
What is the standard identifier format used to uniquely catalogue publicly known information security vulnerabilities across the industry?
 * A) ISO/IEC 27001
 * B) CVE (Common Vulnerabilities and Exposures)
 * C) CVSS (Common Vulnerability Scoring System)
 * D) CERT Advisory
### Question 4 (Domain 2: Vulnerability Scanning)
Which nmap command execution performs a stealthy SYN stealth scan against a target subnet to detect open ports?
 * A) nmap -sT 192.168.1.0/24
 * B) nmap -sS 192.168.1.0/24
 * C) nmap -sU 192.168.1.0/24
 * D) nmap -A 192.168.1.0/24
### Question 5 (Domain 3: Cryptography)
Which cryptographic algorithm is a widely used symmetric block cipher utilizing key lengths of 128, 192, or 256 bits?
 * A) RSA
 * B) ECC
 * C) AES
 * D) Diffie-Hellman
### Question 6 (Domain 3: Hashing Algorithms)
Which hashing algorithm produces a fixed 256-bit (32-byte) hash output and is currently recommended for file integrity verification?
 * A) MD5
 * B) SHA-1
 * C) SHA-256
 * D) CRC32
### Question 7 (Domain 3: PKI)
In a Public Key Infrastructure (PKI), which entity digitally signs and issues X.509 digital certificates to authenticate servers or users?
 * A) Registration Authority (RA)
 * B) Certificate Authority (CA)
 * C) Key Distribution Center (KDC)
 * D) Identity Provider (IdP)
### Question 8 (Domain 3: OpenSSL)
Which OpenSSL command generates a new RSA private key of length 4096 bits?
 * A) openssl genrsa -out private.key 4096
 * B) openssl req -newkey rsa:4096 -nodes -out private.key
 * C) openssl rsa -create 4096 -out private.key
 * D) openssl x509 -genkey rsa:4096 -out private.key
### Question 9 (Domain 4: Storage Security)
Which Linux subsystem provides kernel-level full disk encryption (FDE) by managing block device crypto-mappings?
 * A) AppArmor
 * B) LUKS / dm-crypt
 * C) eCryptfs
 * D) SELinux
### Question 10 (Domain 4: Storage Hardening)
What command initializes a LUKS-encrypted partition on device node /dev/sdb1?
 * A) cryptsetup luksFormat /dev/sdb1
 * B) cryptsetup luksOpen /dev/sdb1 my_encrypted_vol
 * C) mkfs.luks /dev/sdb1
 * D) encryptfs /dev/sdb1
### Question 11 (Domain 5: Netfilter)
In the Netfilter packet flow architecture, which hook handles incoming packets before any routing decisions are made?
 * A) INPUT
 * B) PREROUTING
 * C) FORWARD
 * D) POSTROUTING
### Question 12 (Domain 5: Firewall Rules - nftables)
Which command adds a rule to block incoming TCP traffic destined for port 23 (Telnet) using nftables?
 * A) nft add rule inet filter input tcp dport 23 drop
 * B) iptables -A INPUT -p tcp --dport 23 -j REJECT
 * C) nft insert rule filter output tcp dport 23 deny
 * D) nft drop tcp port 23
### Question 13 (Domain 5: Packet Analysis)
Which tcpdump command captures and displays packets on interface eth0 filtering specifically for HTTP traffic (port 80)?
 * A) tcpdump -i eth0 port 80
 * B) tcpdump -A eth0 tcp --port 80
 * C) tcpdump -w eth0 dst port 80
 * D) tcpdump -i eth0 -p http
### Question 14 (Domain 5: VPN Security)
Which protocol operates at the Network layer (Layer 3) of the OSI model to establish encrypted VPN tunnels between hosts or networks?
 * A) TLS / SSL
 * B) IPsec
 * C) SSH
 * D) HTTPS
### Question 15 (Domain 6: PAM)
Which module type in Pluggable Authentication Modules (PAM) configuration manages account expiration, access hours, and user authorization limits?
 * A) auth
 * B) account
 * C) password
 * D) session
### Question 16 (Domain 6: User Security)
Which file stores salted password digests and account expiration metadata accessible only by administrative accounts?
 * A) /etc/passwd
 * B) /etc/shadow
 * C) /etc/security/pwquality.conf
 * D) /etc/login.defs
### Question 17 (Domain 6: Password Aging)
Which Linux command is used to set the maximum number of days a user's password remains valid?
 * A) passwd -x 90 username
 * B) chage -M 90 username
 * C) usermod -e 90 username
 * D) shadowctl -m 90 username
### Question 18 (Domain 1: Non-Repudiation)
Which security property guarantees that the author of a message or transaction cannot deny having created or transmitted it?
 * A) Confidentiality
 * B) Availability
 * C) Non-repudiation
 * D) Redundancy
### Question 19 (Domain 2: Incident Response)
What is the primary objective of the *Containment* phase in the Incident Response lifecycle?
 * A) Restore system operations back to full capacity.
 * B) Prevent the security breach or infection from spreading across systems.
 * C) Determine the identity of the human attacker.
 * D) Document lessons learned post-incident.
### Question 20 (Domain 3: Diffie-Hellman)
What is the principal purpose of the Diffie-Hellman key exchange protocol?
 * A) Authenticate server digital signatures over public networks.
 * B) Allow two parties to establish a shared secret key over an insecure channel without transmitting the key itself.
 * C) Encrypt large static files stored at rest on secondary block storage.
 * D) Provide non-repudiation for financial transaction logs.
### Question 21 (Domain 4: Hardware Security)
Which hardware component embedded on a motherboard provides tamper-resistant cryptographic storage for platform measurement keys, enabling Secure Boot?
 * A) HSM (Hardware Security Module)
 * B) TPM (Trusted Platform Module)
 * C) DMA Controller
 * D) PCIe Switch
### Question 22 (Domain 5: Network Attacks)
An attacker floods a network switch with spoofed MAC addresses to force the switch into hub-like behavior and capture all traffic. What is this attack called?
 * A) ARP Spoofing
 * B) MAC Flooding
 * C) DNS Poisoning
 * D) Man-in-the-Middle (MitM)
### Question 23 (Domain 6: MFA)
In Time-based One-Time Password (TOTP) implementations, what two inputs generate the temporary one-time passcode?
 * A) Shared Secret Key and Current Timestamp
 * B) Private Key and Sequential Counter
 * C) User Password and HMAC Signature
 * D) Biometric Data and System Time
### Question 24 (Domain 6: Privacy Frameworks)
Under GDPR, what is the term used for any data that directly or indirectly identifies a living individual (e.g., IP address, email, passport number)?
 * A) Intellectual Property (IP)
 * B) Personally Identifiable Information (PII) / Personal Data
 * C) Open Source Intelligence (OSINT)
 * D) Confidential Business Asset
### Question 25 (Domain 2: Penetration Testing vs. Auditing)
How does a penetration test differ fundamentally from a security audit?
 * A) Security audits attempt active system exploitation, whereas penetration tests only review documentation.
 * B) Penetration tests actively attempt to bypass controls and exploit weaknesses, whereas audits assess compliance against documented standards.
 * C) Audits are always conducted by external threat actors.
 * D) Penetration tests do not require permission from system owners.
### Question 26 (Domain 3: Asymmetric Encryption)
If Alice wants to send a confidential, encrypted file to Bob using asymmetric cryptography, which key must Alice use to encrypt the file?
 * A) Alice's Private Key
 * B) Alice's Public Key
 * C) Bob's Public Key
 * D) Bob's Private Key
### Question 27 (Domain 3: Digital Signatures)
If Bob receives a signed file from Alice, which key does Bob use to verify Alice's digital signature?
 * A) Alice's Public Key
 * B) Alice's Private Key
 * C) Bob's Public Key
 * D) Shared Symmetric Key
### Question 28 (Domain 4: Malware Types)
What type of malicious code hides inside a legitimate-looking software executable and installs unauthorized access backdoors?
 * A) Worm
 * B) Trojan Horse
 * C) Rootkit
 * D) Spyware
### Question 29 (Domain 5: Firewall NAT)
In Linux Netfilter, which hook and target combination performs Source NAT (SNAT) or Masquerading for outbound internal LAN traffic?
 * A) PREROUTING chain with DNAT target
 * B) POSTROUTING chain with SNAT or MASQUERADE target
 * C) FORWARD chain with DROP target
 * D) INPUT chain with ACCEPT target
### Question 30 (Domain 6: Social Engineering)
An attacker places targeted phone calls to company executives while impersonating IT Helpdesk support to extract user credentials. What attack vector is being executed?
 * A) Spear Phishing
 * B) Vishing (Voice Phishing)
 * C) Shoulder Surfing
 * D) Waterhole Attack
### Question 31 (Domain 1: Threat Vectors)
What vulnerability occurs when an application accepts user input without proper validation and executes arbitrary SQL statements on the backend database?
 * A) Cross-Site Scripting (XSS)
 * B) SQL Injection (SQLi)
 * C) Buffer Overflow
 * D) Remote Code Execution (RCE)
### Question 32 (Domain 2: CERT Advisories)
What is the primary function of a Computer Emergency Response Team (CERT) advisory?
 * A) Enforce legal prosecution against global cybercriminals.
 * B) Issue technical guidance and mitigation procedures for newly discovered vulnerabilities.
 * C) Sell proprietary antivirus software licenses to enterprise customers.
 * D) Audit corporate accounting systems for compliance.
### Question 33 (Domain 3: Let's Encrypt)
Which protocol is automatically executed by Let's Encrypt clients (such as Certbot) to prove domain ownership and obtain X.509 certificates?
 * A) SCEP
 * B) ACME (Automated Certificate Management Environment)
 * C) OCSP
 * D) EST
### Question 34 (Domain 4: System Auditd)
Which command queries the audit log for all entries recorded under the specific search tag perm_mods?
 * A) auditctl -l perm_mods
 * B) ausearch -k perm_mods
 * C) journalctl -u auditd -k perm_mods
 * D) aureport --key perm_mods
### Question 35 (Domain 5: Wi-Fi Security)
Which Wi-Fi security standard introduces Simultaneous Authentication of Equals (SAE) to protect personal networks against offline dictionary attacks?
 * A) WEP
 * B) WPA-TKIP
 * C) WPA2-PSK
 * D) WPA3-Personal
### Question 36 (Domain 6: Passphrase Policies)
Which configuration parameter in /etc/security/pwquality.conf sets the minimum number of character classes required in a password?
 * A) minlen
 * B) minclass
 * C) dcredit
 * D) lcredit
### Question 37 (Domain 2: Penetration Testing Phases)
During which phase of a formal penetration test does the security engineer gather publicly available intelligence (OSINT) regarding the target organization?
 * A) Exploitation
 * B) Reconnaissance / Footprinting
 * C) Reporting
 * D) Post-Exploitation
### Question 38 (Domain 3: Certificate Revocation)
Which real-time online protocol allows a web browser to check whether an X.509 digital certificate has been revoked by the issuing CA without downloading a complete revocation list?
 * A) CRL
 * B) OCSP (Online Certificate Status Protocol)
 * C) LDAP
 * D) DNSSEC
### Question 39 (Domain 4: Backup Strategies)
In a backup rotation schedule, which backup type copies all data modified since the *last full backup*, without clearing the archive bit?
 * A) Full Backup
 * B) Differential Backup
 * C) Incremental Backup
 * D) Synthetic Backup
### Question 40 (Domain 5: System Logging)
In rsyslog configuration syntax, what does a double at-sign (@@) preceding a remote destination host IP address indicate?
 * A) Forward logs over UDP port 514
 * B) Forward logs over TCP port 514
 * C) Write logs directly to local disk
 * D) Compress logs using Gzip before sending
## 10.3 Answer Key & Detailed Explanations
| Q# | Answer | Domain | Detailed Explanation |
|---|---|---|---|
| **1** | **B** | Domain 1 | Intercepting unencrypted traffic allows unauthorized parties to view sensitive data, directly violating **Confidentiality**. |
| **2** | **C** | Domain 1 | **Advanced Persistent Threats (APTs)** are well-resourced, highly skilled actors often state-sponsored with long-term goals. |
| **3** | **B** | Domain 2 | **CVE (Common Vulnerabilities and Exposures)** provides standardized identifiers (e.g., CVE-2026-12345) for security flaws. |
| **4** | **B** | Domain 2 | The -sS switch in nmap executes a **SYN stealth scan** (half-open scan), which does not complete full TCP handshakes. |
| **5** | **C** | Domain 3 | **AES (Advanced Encryption Standard)** is the standard symmetric block cipher utilizing 128, 192, or 256-bit keys. |
| **6** | **C** | Domain 3 | **SHA-256** produces a 256-bit hash digest and remains the industry standard for secure cryptographic hashing. |
| **7** | **B** | Domain 3 | The **Certificate Authority (CA)** validates identity credentials and issues signed X.509 digital certificates. |
| **8** | **A** | Domain 3 | openssl genrsa -out private.key 4096 generates an unencrypted RSA private key of 4096 bits. |
| **9** | **B** | Domain 4 | **LUKS (Linux Unified Key Setup)** using dm-crypt is the standard Linux kernel mechanism for full disk encryption. |
| **10** | **A** | Domain 4 | cryptsetup luksFormat /dev/sdb1 writes the LUKS header and initializes encrypted storage on the specified block device. |
| **11** | **B** | Domain 5 | Packets pass through the Netfilter **PREROUTING** hook first, making it ideal for Destination NAT (DNAT). |
| **12** | **A** | Domain 5 | nft add rule inet filter input tcp dport 23 drop is the correct nftables syntax to block TCP port 23 traffic. |
| **13** | **A** | Domain 5 | tcpdump -i eth0 port 80 captures and filters packets specifically on interface eth0 matching port 80. |
| **14** | **B** | Domain 5 | **IPsec** operates at Layer 3 (Network layer), providing network-wide encryption and authentication tunnels. |
| **15** | **B** | Domain 6 | The **account** PAM module type handles non-authentication restrictions, such as account expiration and access hours. |
| **16** | **B** | Domain 6 | **/etc/shadow** stores password digests, salts, and account aging parameters restricted exclusively to root permissions. |
| **17** | **B** | Domain 6 | chage -M <days> <user> sets the maximum number of days between mandatory password changes. |
| **18** | **C** | Domain 1 | **Non-repudiation** guarantees that an author or sender cannot deny the authenticity or origin of a signed message. |
| **19** | **B** | Domain 2 | The **Containment** phase focuses on isolating compromised assets to prevent an attack from spreading further. |
| **20** | **B** | Domain 3 | **Diffie-Hellman** allows two parties to securely generate a shared secret key over an unencrypted network channel. |
| **21** | **B** | Domain 4 | A **Trusted Platform Module (TPM)** is a motherboard chip providing secure cryptographic hardware key storage. |
| **22** | **B** | Domain 4 | **MAC Flooding** fills a switch's CAM table with fake addresses, causing it to fail open and broadcast all traffic like a hub. |
| **23** | **A** | Domain 6 | **TOTP** (RFC 6238) uses a shared secret key combined with the current time step window to generate passcodes. |
| **24** | **B** | Domain 6 | **PII / Personal Data** refers to any data that can uniquely identify an individual under data privacy laws like GDPR. |
| **25** | **B** | Domain 2 | Penetration tests actively exploit vulnerabilities, whereas audits assess system compliance against baseline policies. |
| **26** | **C** | Domain 3 | To ensure only Bob can decrypt the message, Alice must encrypt it using **Bob's Public Key**. |
| **27** | **A** | Domain 3 | Digital signatures are created using the sender's private key and verified by anyone using **Alice's Public Key**. |
| **28** | **B** | Domain 4 | A **Trojan Horse** masquerades as legitimate software while concealing malicious payload functionality. |
| **29** | **B** | Domain 5 | Source NAT (SNAT) and Masquerading alter the source IP address on outbound packets in the **POSTROUTING** chain. |
| **30** | **B** | Domain 6 | **Vishing** (voice phishing) uses phone calls and pretexting to trick victims into revealing sensitive credentials. |
| **31** | **B** | Domain 1 | **SQL Injection (SQLi)** occurs when unvalidated user input is executed directly by a backend SQL engine. |
| **32** | **B** | Domain 2 | **CERT Advisories** provide public technical analysis, threat warnings, and remediation guidance for software flaws. |
| **33** | **B** | Domain 3 | Let's Encrypt relies on the **ACME (Automated Certificate Management Environment)** protocol for automated issuance. |
| **34** | **B** | Domain 4 | ausearch -k perm_mods searches the audit log (/var/log/audit/audit.log) for events assigned the key perm_mods. |
| **35** | **D** | Domain 5 | **WPA3-Personal** uses SAE (Simultaneous Authentication of Equals) to mitigate offline brute-force dictionary attacks. |
| **36** | **B** | Domain 6 | minclass in /etc/security/pwquality.conf specifies the required minimum number of character classes. |
| **37** | **B** | Domain 2 | The initial **Reconnaissance / Footprinting** phase gathers intelligence and OSINT prior to active exploitation. |
| **38** | **B** | Domain 3 | **OCSP (Online Certificate Status Protocol)** provides real-time digital certificate validity checks without downloading large CRLs. |
| **39** | **B** | Domain 4 | A **Differential Backup** copies all files that have changed since the last *full* backup without clearing the archive flags. |
| **40** | **B** | Domain 5 | In rsyslog.conf, @ indicates UDP log forwarding, while @@ specifies reliable **TCP log forwarding**. |
## 10.4 Final Hands-On Capstone Project
### Scenario Objective
You are completing the final deployment of a secure Linux node for **Cybergate Services Private Limited**. Apply the full security stack covered across this guide by completing the following sequence:
 1. **Storage Hardening:** Verify or simulate a encrypted block device mapping.
 2. **System Auditing:** Establish an active audit watch on /etc/shadow with key tag shadow_monitor.
 3. **Network Firewall:** Configure an nftables baseline dropping incoming TCP port 23 traffic while allowing established traffic.
 4. **PAM Enforcement:** Verify password aging param
