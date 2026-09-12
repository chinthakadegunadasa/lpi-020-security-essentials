# Chapter 9: Identity & Privacy Protection
## Objective Overview: LPI 020 Security Essentials – Identity & Privacy Protection
| Field | Details |
|---|---|
| **Objective Domain** | Topic 6: Identity & Privacy Protection |
| **Weight** | 3 |
| **Description** | Understand identity management models, Multi-Factor Authentication (MFA), Single Sign-On (SSO), passphrase security, password managers, social engineering vector mitigations, data privacy frameworks (GDPR), and information classification policies. |
| **Key Knowledge Areas** | Authentication Models (MFA, TOTP, HOTP, FIDO2/WebAuthn), SSO Mechanisms (SAML, OAuth2, OIDC), Linux PAM Architecture, Passphrase Policy Hardening (pam_pwquality), Shadow Password Database (/etc/shadow), Social Engineering Vectors (Phishing, Pretexting, Spear Phishing), Data Privacy (GDPR, Data Protection Rights, Information Classification). |
## 9.1 Identity Management and Authentication Models
Identity, Authentication, and Authorization form the foundation of access management:
```
+-------------------------------------------------------------------------+
|                  AUTHENTICATION & ACCESS CONTROL                        |
+-------------------------------------------------------------------------+
                                     |
    +--------------------------------+--------------------------------+
    |                                |                                |
    v                                v                                v
+-----------------------+  +-----------------------+  +-----------------------+
| SOMETHING YOU KNOW    |  | SOMETHING YOU HAVE    |  | SOMETHING YOU ARE     |
| (Passwords, PINs)     |  | (TOTP Tokens, FIDO2)  |  | (Biometrics, Retina)  |
+-----------------------+  +-----------------------+  +-----------------------+

```
### 1. Multi-Factor Authentication (MFA)
MFA requires presenting two or more distinct authentication factors:
 * **Something You Know:** Passwords, passphrases, or PINs.
 * **Something You Have:** Hardware security keys (FIDO2/YubiKey), Time-based One-Time Password (**TOTP**) authenticators, or smart cards.
 * **Something You Are:** Biometric attributes (fingerprint scan, facial recognition).
### 2. Single Sign-On (SSO) Protocols
Federated identity protocols allow users to authenticate once and access multiple independent software systems:
 * **OAuth 2.0:** An open authorization framework delegated to grant third-party applications limited access to user resources.
 * **OpenID Connect (OIDC):** An identity layer built directly on top of OAuth 2.0, providing authentication via JWT (JSON Web Tokens).
 * **SAML 2.0:** An XML-based standard for exchanging authentication and authorization data between an Identity Provider (IdP) and a Service Provider (SP), heavily used in corporate environments.
## 9.2 Linux Authentication Architecture: PAM & Shadow Files
Linux manages authentication dynamically through **Pluggable Authentication Modules (PAM)** and stores password digests in secure system databases.
```
+-----------------------------------------------------------------------------------+
|                        PLUGGABLE AUTHENTICATION MODULES (PAM)                     |
+-----------------------------------------------------------------------------------+

 Application (sshd, login, sudo)
         |
         v
 [ PAM Stack ] ---> Module Types:
                      ├── auth       (Verifies user identity)
                      ├── account    (Checks account validity, expiration, access hours)
                      ├── password   (Handles password changes & complexity enforcement)
                      └── session    (Configures environment pre/post login)

```
### 1. User Password Storage (/etc/passwd vs. /etc/shadow)
 * /etc/passwd: Publicly readable system mapping of usernames, UIDs, GIDs, home directories, and default shells.
 * /etc/shadow: Restricted file (readable only by root or group shadow) containing salted password cryptographic digests and account expiration parameters.
```
[Format of /etc/shadow]
username:$id$salt$hashed_password:lastchange:min:max:warn:inact:expire

```
 * **$id$ Identifier:** Specifies the hashing algorithm used. For example, $6$ indicates **SHA-512**, and $y$ or $argon2id$ indicates modern **Argon2id**.
## 9.3 Data Privacy Rights & Information Classification
Modern security engineers must comply with legal frameworks governing personal data protection:
 * **General Data Protection Regulation (GDPR):** Enforces data privacy rights for individuals, including the **Right to Access**, **Right to Erasure** (Right to be Forgotten), and strict guidelines for processing **Personally Identifiable Information (PII)**.
 * **Information Classification Labels:**
   * **Public:** Information approved for unrestricted public distribution.
   * **Internal:** Information restricted to organizational personnel.
   * **Confidential:** Sensitive business operational data restricted to specific groups.
   * **Restricted / Top Secret:** High-risk trade secrets, PII, or critical cryptographic keys where unauthorized exposure causes severe damage.
## 9.4 Practical Scenario: Hardening System Passwords & User Account Audits
### Scenario Context
You are an Infrastructure Security Specialist. Following an enterprise audit, you are tasked with enforcing strict password complexity rules on a Linux server using PAM, configuring account lockouts after repeated failed logins (pam_faillock), enforcing password expiration parameters, and configuring two-factor authentication (TOTP) for local SSH access.
```
[ Application / SSH Login ]
            |
            v
 [ PAM authentication stack ] ---> [ pam_pwquality: Enforce Complexity ]
            |                      [ pam_faillock: Lock on 3 Failures  ]
            v                      [ pam_google_authenticator: TOTP    ]
 [ Authorized Shell Access ]

```
## 9.5 Hands-On Laboratory: Configuring PAM Password Hardening and TOTP 2FA
In this lab, you will configure PAM password complexity rules, enforce password aging policies using chage, analyze the shadow database, and set up a TOTP two-factor authentication mechanism.
### Prerequisites
 * A Linux host (Ubuntu/Debian or RHEL/Rocky Linux) with root privileges (sudo).
 * Installed utilities: libpam-pwquality (or pam_pwquality), libpam-google-authenticator (or google-authenticator).
```bash
# Ubuntu / Debian installation
sudo apt-get update && sudo apt-get install -y libpam-pwquality libpam-google-authenticator

```
### Step 1: Inspecting User Accounts and /etc/shadow Structures
 1. Inspect the standard account file entry for a test user:
   ```bash
   grep "^$USER" /etc/passwd
   
   ```
 2. Inspect the encrypted entry in /etc/shadow (requires elevated privileges):
   ```bash
   sudo grep "^$USER" /etc/shadow
   
   ```
   * *Identify the hash algorithm field (e.g., $6$ for SHA-512 or $y$ for Yescrypt).*
 3. Display the detailed password aging parameters for your user account using chage:
   ```bash
   sudo chage -l $USER
   
   ```
 4. Enforce mandatory password expiration rules (e.g., maximum password age of 90 days, minimum 7 days before change, 14-day warning period):
   ```bash
   sudo chage -M 90 -m 7 -W 14 $USER
   
   ```
 5. Verify updated expiration policies:
   ```bash
   sudo chage -l $USER | grep -E "(Maximum|Minimum|Warning)"
   
   ```
### Step 2: Enforcing Password Complexity via PAM (pam_pwquality)
 1. Edit the PAM password quality configuration file /etc/security/pwquality.conf using your preferred editor:
   ```bash
   sudo nano /etc/security/pwquality.conf
   
   ```
 2. Set the following mandatory security policies:
   ```text
   minlen = 14
   minclass = 3
   maxrepeat = 2
   gecoscheck = 1
   remember = 5
   
   ```
   * *Parameter breakdown:*
     * minlen = 14: Enforces a minimum password length of 14 characters.
     * minclass = 3: Requires characters from at least 3 distinct classes (uppercase, lowercase, digits, special characters).
     * maxrepeat = 2: Allows a maximum of 2 consecutive identical characters.
     * gecoscheck = 1: Prevents including words present in the user's GECOS field (e.g., real name).
 3. Create a temporary unprivileged test user to test password compliance:
   ```bash
   sudo useradd -m -s /bin/bash testsecuser
   
   ```
 4. Set a non-compliant password for testsecuser to verify PAM rejection rules:
   ```bash
   sudo passwd testsecuser
   
   ```
   * *Try simple passwords like Password123 or aaa111 to confirm rejection by pam_pwquality.*
### Step 3: Setting Up Time-Based One-Time Password (TOTP) MFA
 1. Switch to the unprivileged test user account:
   ```bash
   sudo su - testsecuser
   
   ```
 2. Initialize the Google Authenticator TOTP generator:
   ```bash
   google-authenticator
   
   ```
 3. Respond to the initialization prompts:
   * **Make tokens time-based?** y
   * **Update .google_authenticator file?** y
   * **Disallow multiple uses of the same token?** y
   * **Increase time skew window?** n
   * **Enable rate-limiting?** y
 4. *Observe the emergency scratch codes and secret key output generated in the terminal session.*
 5. Exit back to your primary administrative user account:
   ```bash
   exit
   
   ```
### Step 4: Configuring PAM for SSH Multi-Factor Authentication
 1. Configure PAM to require the TOTP module for SSH authentication by editing /etc/pam.d/sshd:
   ```bash
   sudo nano /etc/pam.d/sshd
   
   ```
 2. Add the following line to the top of the authentication section:
   ```text
   auth required pam_google_authenticator.so nullok
   
   ```
   * *Note:* The nullok flag ensures users without a configured TOTP key can still log in using standard authentication methods during migration.
 3. Enable multi-factor authentication in the SSH daemon configuration (/etc/ssh/sshd_config):
   ```bash
   sudo nano /etc/ssh/sshd_config
   
   ```
 4. Ensure or update the following directives:
   ```text
   KbdInteractiveAuthentication yes
   UsePAM yes
   
   ```
 5. Restart the SSH daemon service to apply changes:
   ```bash
   sudo systemctl restart ssh
   
   ```
### Laboratory Cleanup
Remove the test user account and configuration artifacts created during this lab session:
```bash
sudo userdel -r testsecuser 2>/dev/null || true

```
## 9.6 Chapter Review & Knowledge Check
### Self-Assessment Questions
#### Question 1
Which file on a standard Linux system contains user account salt parameters and password hashes restricted exclusively to administrative read access?
 * A) /etc/passwd
 * B) /etc/security/pwquality.conf
 * C) /etc/shadow
 * D) /etc/gshadow
#### Question 2
What type of authentication factor does a Time-based One-Time Password (TOTP) token generated by an authenticator application represent?
 * A) Something You Know
 * B) Something You Have
 * C) Something You Are
 * D) Somewhere You Are
#### Question 3
Under the General Data Protection Regulation (GDPR), which data privacy principle mandates that users possess the right to request the complete deletion of their personal data from enterprise systems?
 * A) Right to Portability
 * B) Right to Erasure (Right to be Forgotten)
 * C) Right to Rectification
 * D) Right to Data Minimization
#### Question 4
Which CLI utility allows an administrator to set password aging rules, such as maximum lifetime and mandatory warning periods, for Linux accounts?
 * A) pam_pwquality
 * B) chage
 * C) shadowconfig
 * D) usermod
### Answers and Explanations
 1. **Correct Answer: C (/etc/shadow)**
   * *Explanation:* /etc/shadow contains sensitive password hashes and expiration metadata, restricted to root access to protect against offline password cracking attacks.
 2. **Correct Answer: B (Something You Have)**
   * *Explanation:* A TOTP authenticator app or hardware token represents a physical or digital asset in your possession ("Something You Have").
 3. **Correct Answer: B (Right to Erasure / Right to be Forgotten)**
   * *Explanation:* Article 17 of GDPR establishes the Right to Erasure, allowing data subjects to request deletion of their personal data under specific conditions.
 4. **Correct Answer: B (chage)**
   * *Explanation:* The chage (change age) command modifies user password expiration and aging parameters stored in /etc/shadow.
   
