
# Chapter 4: Cryptographic Primitives and PKI Systems
## Objective Overview: LPI 022.1 Cryptography and Public Key Infrastructure
| Field | Details |
|---|---|
| **Objective Code** | 022.1 |
| **Weight** | 3 |
| **Description** | Understand symmetric, asymmetric, and hybrid cryptography, hashing algorithms, digital signatures, and key exchange algorithms. Understand Public Key Infrastructures (PKI), Certificate Authorities (CAs), X.509 certificate formats, lifecycle management, and Let's Encrypt workflows. |
| **Key Knowledge Areas** | Symmetric vs. Asymmetric Cryptography, Hash Functions (SHA-256, MD5), Key Exchange (Diffie-Hellman), Hybrid Encryption, Perfect Forward Secrecy (PFS), End-to-End vs. Transport Encryption, PKI & Trust Chains, X.509 Fields (Subject, Issuer, Validity), CSR Generation, Certificate Revocation (CRL/OCSP), OpenSSL CLI. |
## 4.1 Cryptographic Primitives and Architectural Concepts
Cryptography forms the foundation of modern data protection, privacy, and identity verification across digital infrastructures.
```
+-------------------------------------------------------------------------+
|                        CRYPTOGRAPHIC MECHANISMS                         |
+-------------------------------------------------------------------------+
                                     |
    +--------------------------------+--------------------------------+
    |                                |                                |
    v                                v                                v
+-----------------------+  +-----------------------+  +-----------------------+
|  SYMMETRIC CIPHERS    |  |  ASYMMETRIC CIPHERS   |  |    HASH FUNCTIONS     |
| (AES, ChaCha20)       |  | (RSA, ECC, Ed25519)   |  | (SHA-256, SHA-512)    |
| - Single Shared Key   |  | - Key Pair (Pub/Priv) |  | - One-way Digest      |
| - High Speed / Bulk   |  | - Identity & Key Ex.  |  | - Integrity Checks    |
+-----------------------+  +-----------------------+  +-----------------------+

```
### 1. Symmetric Encryption
Uses a single shared secret key for both data encryption and decryption.
 * **Characteristics:** Extremely fast computational throughput; ideal for bulk data storage and network stream payload encryption.
 * **Common Algorithms:** Advanced Encryption Standard (**AES**-128, AES-256), **ChaCha20**.
 * **Key Challenge:** Securely exchanging the shared secret key across an untrusted network without interception.
### 2. Asymmetric Encryption (Public Key Cryptography)
Uses a mathematically linked key pair: a **Public Key** (freely shared) and a **Private Key** (kept strictly secret).
 * **Data Encryption Flow:** Data encrypted with the recipient's *Public Key* can only be decrypted by the recipient's matching *Private Key*.
 * **Digital Signatures Flow:** Data signed with the sender's *Private Key* can be verified by anyone holding the sender's *Public Key*, guaranteeing authenticity and **Non-Repudiation**.
 * **Common Algorithms:** **RSA**, Elliptic Curve Cryptography (**ECC** / ECDSA), **Ed25519**.
### 3. Hash Functions and Key Exchanges
 * **Cryptographic Hash Functions:** One-way algorithms that transform arbitrary inputs into a fixed-length string digest. Used for integrity validation (sha256sum). Legacy hashes like **MD5** and **SHA-1** are cryptographically broken and deprecated.
 * **Diffie–Hellman (DH) Key Exchange:** A method allowing two parties to negotiate a shared symmetric key over an insecure network channel without transmitting the key itself.
 * **Perfect Forward Secrecy (PFS):** A feature of key exchange protocols ensuring that even if a server's long-term private key is compromised in the future, past session traffic cannot be decrypted. Ephemeral Diffie-Hellman (ECDHE) provides PFS.
### 4. Hybrid Cryptography
Modern production security protocols (e.g., TLS, SSH, PGP) combine symmetric and asymmetric ciphers:
 1. Asymmetric cryptography (RSA/ECC) or Key Exchange (Diffie-Hellman) authenticates identity and establishes a secure session key.
 2. High-speed symmetric cryptography (AES-GCM or ChaCha20) encrypts the bulk application data stream.
```
[Sender]                                                        [Recipient]
   |                                                                 |
   |-- 1. Generate Random Session Key (Symmetric AES) -------------->|
   |-- 2. Encrypt Session Key with Recipient's Public Key --------->| (Decrypts Key via Private Key)
   |                                                                 |
   |<================ 3. Bulk Data Channel =========================>|
   |            Encrypted via Shared Symmetric Key                   |

```
### 5. Transport Encryption vs. End-to-End Encryption (E2EE)
 * **Transport Encryption (TLS):** Encrypts data in transit between the client and the intermediate server/service provider. Data is decrypted in memory at the relay node.
 * **End-to-End Encryption (E2EE):** Data is encrypted at the source device and remains encrypted through all intermediate relays, unreadable by service providers, and decrypted only on the target destination device.
## 4.2 Public Key Infrastructure (PKI) and X.509 Digital Certificates
**Public Key Infrastructure (PKI)** encompasses the hardware, software, policies, and roles required to create, manage, distribute, use, store, and revoke digital certificates.
```
               +----------------------------------+
               |        Trusted Root CA           |
               | (Self-Signed Root Certificate)   |
               +----------------+-----------------+
                                |
                                v
               +----------------------------------+
               |       Intermediate CA            |
               |  (Signed by Trusted Root CA)     |
               +----------------+-----------------+
                                |
                                v
               +----------------------------------+
               |      Leaf / End-Entity Cert      |
               |  (Issued to domain.com / server) |
               +----------------------------------+

```
### Trust Chains and Certificate Authorities (CAs)
 * **Root Certificate Authority (Root CA):** The ultimate trust anchor. Root CA certificates are self-signed and pre-installed into operating system and browser trust stores.
 * **Intermediate Certificate Authority:** Issued by the Root CA to handle day-to-day certificate signing, protecting the offline Root CA private key from exposure.
 * **Leaf (End-Entity) Certificate:** Issued to specific domain names, servers, or email addresses.
### Structure of an X.509 Certificate
An X.509 standard digital certificate contains public keys alongside identity attributes verified by a CA:
 * **Subject:** The entity (domain name, organization) to whom the certificate is issued (e.g., CN=[www.example.com](https://www.example.com)).
 * **Issuer:** The CA that signed and validated the certificate.
 * **Validity:** Effective date ranges (Not Before and Not After).
 * **Subject Alternative Name (SAN):** Modern X.509 extension specifying all domain names, subdomains, or IP addresses secured by the single certificate.
 * **Public Key:** The public key material and signature algorithm (e.g., RSA 4096-bit or ECDSA secp384r1).
### Certificate Lifecycle and Revocation
 * **Certificate Signing Request (CSR):** A cryptographically signed block of text generated by an applicant containing their Public Key and organizational information sent to a CA for signing.
 * **Certificate Revocation List (CRL):** A published list of revoked certificate serial numbers signed by the CA.
 * **Online Certificate Status Protocol (OCSP):** A real-time network protocol used by clients to query CA status servers regarding certificate validity.
 * **Let's Encrypt / ACME:** An automated, non-profit Certificate Authority providing free Domain Validated (DV) X.509 certificates via the Automated Certificate Management Environment (ACME) protocol.
## 4.3 Practical Scenario: Enterprise Public Key Infrastructure Deployment
### Scenario Context
You are a Lead Infrastructure Engineer at **Vortex Micro-Grid Systems**. The enterprise internal web architecture requires upgrading internal microservice communications from unencrypted HTTP to secure HTTPS. You must establish a local Private Certificate Authority (CA) structure, issue signed X.509 leaf certificates to internal edge nodes, and test cryptographic trust chains.
```
[Private Root CA] ---> Signs ---> [Internal Server CSR] ---> Issues ---> [Server X.509 Cert]

```
### Implementation Execution Plan
 1. **Private Root CA Generation:** Construct a 4096-bit RSA private key and self-signed Root CA certificate with a 10-year validity window.
 2. **Server CSR Generation:** Generate a 2048-bit server private key and issue a Certificate Signing Request (CSR) specifying internal domain attributes.
 3. **CA Signature Execution:** Sign the server CSR using the Root CA key, enforcing X.509 v3 SAN extensions.
 4. **Verification & Audit:** Validate the resulting certificate against the CA public key trust store using standard OpenSSL verification tools.
## 4.4 Hands-On Laboratory: Generating PKI Keys, CSRs, and X.509 Certificates with OpenSSL
In this lab, you will use the openssl CLI command to act as both a Certificate Authority and an enterprise system administrator to issue and inspect X.509 certificates.
### Prerequisites
A Linux terminal with openssl installed (sudo apt install openssl or sudo dnf install openssl).
### Step 1: Creating a Private Certificate Authority (Root CA)
 1. Create a dedicated directory for local PKI operations:
   ```bash
   mkdir -p ~/pki_lab && cd ~/pki_lab
   
   ```
 2. Generate a high-entropy 4096-bit RSA private key for your Root CA:
   ```bash
   openssl genrsa -out rootCA.key 4096
   chmod 600 rootCA.key
   
   ```
 3. Generate a self-signed X.509 Root CA Certificate:
   ```bash
   openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 3650 \
     -out rootCA.crt \
     -subj "/C=LK/ST=Western/L=Colombo/O=Vortex Grid/OU=Security/CN=Vortex Internal Root CA"
   
   ```
 4. Inspect the generated Root CA Certificate parameters:
   ```bash
   openssl x509 -in rootCA.crt -text -noout | head -n 15
   
   ```
### Step 2: Generating Server Keys and a Certificate Signing Request (CSR)
 1. Generate a private key for an internal web application server (appserver.internal):
   ```bash
   openssl genrsa -out appserver.key 2048
   chmod 600 appserver.key
   
   ```
 2. Create a Certificate Signing Request (CSR) for the server:
   ```bash
   openssl req -new -key appserver.key -out appserver.csr \
     -subj "/C=LK/ST=Western/L=Colombo/O=Vortex Grid/OU=IT/CN=appserver.internal"
   
   ```
 3. Verify the CSR contents and public key info:
   ```bash
   openssl req -in appserver.csr -text -noout | grep -E "(Subject:|Public Key:)"
   
   ```
### Step 3: Signing the CSR with the Root CA to Issue the X.509 Certificate
Modern browsers and security clients require the subjectAltName (SAN) extension to be present on leaf certificates.
 1. Create an OpenSSL extension configuration file v3.ext:
   ```bash
   cat << 'EOF' > v3.ext
   authorityKeyIdentifier=keyid,issuer
   basicConstraints=CA:FALSE
   keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
   subjectAltName = @alt_names
   
   [alt_names]
   DNS.1 = appserver.internal
   DNS.2 = *.appserver.internal
   IP.1 = 192.168.1.50
   EOF
   
   ```
 2. Sign the CSR using the Root CA key and certificate:
   ```bash
   openssl x509 -req -in appserver.csr -CA rootCA.crt -CAkey rootCA.key \
     -CAcreateserial -out appserver.crt -days 365 -sha256 -extfile v3.ext
   
   ```
 3. Inspect the newly issued leaf certificate Subject, Issuer, Validity, and SAN fields:
   ```bash
   openssl x509 -in appserver.crt -text -noout | grep -A 2 -E "(Issuer:|Subject:|Not Before|Subject Alternative Name)"
   
   ```
### Step 4: Validating Certificate Trust Chains and Fingerprints
 1. Verify that the server certificate successfully chains back to your local Root CA:
   ```bash
   openssl verify -CAfile rootCA.crt appserver.crt
   
   ```
   * **Expected Output:** appserver.crt: OK
 2. Calculate the SHA-256 fingerprint digest of the certificate:
   ```bash
   openssl x509 -in appserver.crt -noout -fingerprint -sha256
   
   ```
### Step 5: Practical Benchmarking – Symmetric vs. Asymmetric Performance
Execute the built-in OpenSSL speed benchmark to compare symmetric cipher throughput against asymmetric RSA operations:
 1. Test symmetric AES-256-GCM performance:
   ```bash
   openssl speed aes-256-gcm
   
   ```
 2. Test asymmetric RSA signature/verification performance:
   ```bash
   openssl speed rsa2048
   
   ```
   * **Key Takeaway:** Observe how symmetric encryption processes hundreds of megabytes per second, whereas asymmetric operations are significantly slower—illustrating why **Hybrid Cryptography** is universally used in production networks.
## 4.5 Chapter Review & Knowledge Check
### Self-Assessment Questions
#### Question 1
Why do modern security protocols like TLS utilize hybrid cryptography instead of relying exclusively on asymmetric encryption for data transfer?
 * A) Asymmetric encryption algorithms do not support network transport layer protocols.
 * B) Asymmetric encryption is computationally intensive and drastically slower for bulk data transfer compared to symmetric ciphers.
 * C) Symmetric ciphers do not require secret keys to encrypt data streams.
 * D) Asymmetric cryptography cannot generate digital signatures over network channels.
#### Question 2
Which feature of modern key exchange protocols ensures that historic encrypted session traffic remains secure even if the server's long-term private key is compromised in the future?
 * A) Certificate Revocation Lists (CRL)
 * B) Subject Alternative Name (SAN)
 * C) Perfect Forward Secrecy (PFS)
 * D) Data Encryption Standard (DES)
#### Question 3
Which specific field within an X.509 version 3 digital certificate is mandatory for modern web browsers to validate multiple domain names or subdomains assigned to a single host?
 * A) Serial Number
 * B) Subject Alternative Name (SAN)
 * C) Key Identifier Algorithm
 * D) Signature Hash Algorithm
#### Question 4
What is the primary operational role of a Certificate Signing Request (CSR)?
 * A) To securely export a CA's private key to a client system.
 * B) To submit a public key and identity data to a Certificate Authority for digital signing.
 * C) To automatically revoke expired SSL/TLS certificates across browser trust stores.
 * D) To negotiate symmetric AES keys between a client and a web server during a TLS handshake.
### Answers and Explanations
 1. **Correct Answer: B (Asymmetric encryption is computationally intensive and drastically slower for bulk data transfer compared to symmetric ciphers)**
   * *Explanation:* Asymmetric cryptography involves complex mathematical operations (large prime factoring or discrete logarithms), making it inefficient for bulk throughput. Hybrid schemes use asymmetric methods only for key negotiation and identity validation, delegating payload encryption to fast symmetric ciphers (AES).
 2. **Correct Answer: C (Perfect Forward Secrecy - PFS)**
   * *Explanation:* PFS relies on ephemeral key exchange mechanisms (such as ECDHE) so that each session generates unique session keys not derived from the server's master private key.
 3. **Correct Answer: B (Subject Alternative Name - SAN)**
   * *Explanation:* The SAN extension allows an X.509 certificate to specify multiple hostnames, wildcards, or IP addresses protected by the single certificate.
 4. **Correct Answer: B (To submit a public key and identity data to a Certificate Authority for digital signing)**
   * *Explanation:* A CSR contains the applicant's public key and organizational identifier, which the Certificate Authority validates and signs to produce the final X.509 certificate.
   
