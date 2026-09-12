
# Chapter 12: Network Services and Remote Management Architecture
## 12.1 Core Network Application Protocols and Architecture
Enterprise Linux systems rely on foundational network daemons to manage host discovery, automated IP allocation, and secure remote administration. Understanding the architecture of key network daemons, systemd unit dependencies, and security boundaries is essential for operating production systems.
```
                  ENTERPRISE NETWORK SERVICES ARCHITECTURE

   +--------------------------------------------------------------------+
   |                       Client Host Request                          |
   +--------------------------------------------------------------------+
           |                                 |                  |
           v                                 v                  v
     [ DHCP Request ]                 [ DNS Lookup ]      [ SSH Connection ]
   (UDP Ports 67 / 68)                 (Port 53 TCP/UDP)     (Port 22 TCP)
           |                                 |                  |
           v                                 v                  v
+-----------------------+         +--------------------+ +--------------------+
|     isc-dhcp-server   |         |     bind9 / named  | |       sshd         |
| /etc/dhcp/dhcpd.conf  |         | /etc/bind/named.conf| | /etc/ssh/sshd_config|
+-----------------------+         +--------------------+ +--------------------+
           |                                 |                  |
           +---------------------------------+------------------+
                                             |
                                             v
                             [ systemd Service Manager ]
                             [ nftables / UFW Firewall ]

```
### Protocol Overview and Port Bindings
| Service | Protocol | Default Port(s) | Key Daemon / Package | Primary Function & Configuration File |
|---|---|---|---|---|
| **DNS** | Domain Name System | 53 (UDP/TCP) | bind9 (named) | Hostname-to-IP resolution; /etc/bind/named.conf |
| **DHCP** | Dynamic Host Configuration Protocol | 67 (Server), 68 (Client) UDP | isc-dhcp-server / dnsmasq | Automated network parameter assignment; /etc/dhcp/dhcpd.conf |
| **SSH** | Secure Shell | 22 (TCP) | openssh-server (sshd) | Encrypted remote shell access & execution; /etc/ssh/sshd_config |
## 12.2 DNS Infrastructure Configuration with BIND9
The Domain Name System (DNS) translates human-readable domain names into IP addresses. In enterprise environments, BIND9 (named) provides authoritative and recursive name resolution services.
### Key DNS Record Types
 * **A Record:** Maps a hostname to an IPv4 address.
 * **AAAA Record:** Maps a hostname to an IPv6 address.
 * **PTR Record:** Maps an IP address to a hostname (Reverse DNS).
 * **CNAME Record:** Creates an alias pointing one domain name to another canonical name.
 * **NS Record:** Specifies the authoritative name server for a domain zone.
 * **MX Record:** Directs mail traffic to designated mail servers.
### BIND9 Configuration Hierarchy (/etc/bind/)
 * /etc/bind/named.conf: Master configuration entry point.
 * /etc/bind/named.conf.options: Global operational settings, forwarders, and access control lists (ACLs).
 * /etc/bind/named.conf.local: Zone declarations for primary and secondary domains.
## 12.3 Automated IP Allocation using DHCP Services
DHCP dynamically manages network configuration parameters for endpoints, dispensing IP addresses, network masks, default gateways, and DNS servers over a specified lease duration.
### The DHCP DORA Process
 1. **Discover:** Client broadcasts a DHCPDISCOVER packet on UDP port 67 to locate active DHCP servers.
 2. **Offer:** DHCP server responds with a DHCPOFFER unicast/broadcast containing an available IP address.
 3. **Request:** Client returns a DHCPREQUEST packet accepting the offered IP lease.
 4. **Acknowledge:** Server transmits a DHCPACK confirming lease allocation and binding parameters.
## 12.4 Enterprise SSH Hardening and Remote Access
The OpenSSH daemon (sshd) provides cryptographic remote shell access. Hardening SSH access is critical for defending systems against automated brute-force attempts and unauthorized access.
### Critical sshd_config Directives (/etc/ssh/sshd_config)
```ini
# Core SSH Hardening Policy
Port 2222
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 2
AllowUsers adminuser sysadmin

```
## 12.5 Hands-On Lab Framework: Deploying & Securing Core Network Infrastructure Services
### Lab Prerequisites & Setup
 * **Operating System:** Ubuntu 24.04 LTS or RHEL 9 system with sudo privileges.
 * **Network Interfaces:** Primary static IP configured on 192.168.50.10/24.
 * **Required Packages:** bind9, bind9utils, isc-dhcp-server, openssh-server, net-tools.
### Step 1: Install and Configure Authoritative BIND9 DNS
 1. Install BIND9 tools and utilities:
   ```bash
   sudo apt update && sudo apt install -y bind9 bind9utils bind9-doc
   
   ```
 2. Define a local DNS zone in /etc/bind/named.conf.local:
   ```bash
   sudo tee /etc/bind/named.conf.local << 'EOF'
   zone "lab.internal" {
       type master;
       file "/etc/bind/zones/db.lab.internal";
   };
   
   zone "50.168.192.in-addr.arpa" {
       type master;
       file "/etc/bind/zones/db.192.168.50";
   };
   EOF
   
   ```
 3. Create the zone directory and master zone file for lab.internal:
   ```bash
   sudo mkdir -p /etc/bind/zones
   sudo tee /etc/bind/zones/db.lab.internal << 'EOF'
   $TTL    86400
   @       IN      SOA     ns1.lab.internal. admin.lab.internal. (
                              2026091201 ; Serial
                                  604800 ; Refresh
                                   86400 ; Retry
                                 2419200 ; Expire
                                   86400 ) ; Negative Cache TTL
   ;
   @       IN      NS      ns1.lab.internal.
   ns1     IN      A       192.168.50.10
   srv01   IN      A       192.168.50.20
   web     IN      CNAME   srv01.lab.internal.
   EOF
   
   ```
 4. Create the reverse lookup zone file:
   ```bash
   sudo tee /etc/bind/zones/db.192.168.50 << 'EOF'
   $TTL    86400
   @       IN      SOA     ns1.lab.internal. admin.lab.internal. (
                              2026091201 ; Serial
                                  604800 ; Refresh
                                   86400 ; Retry
                                 2419200 ; Expire
                                   86400 ) ; Negative Cache TTL
   ;
   @       IN      NS      ns1.lab.internal.
   10      IN      PTR     ns1.lab.internal.
   20      IN      PTR     srv01.lab.internal.
   EOF
   
   ```
 5. Validate BIND9 configuration syntax and reload the service:
   ```bash
   sudo named-checkconf
   sudo named-checkzone lab.internal /etc/bind/zones/db.lab.internal
   sudo systemctl restart bind9
   sudo systemctl status bind9 --no-pager
   
   ```
 6. Test local DNS resolution using dig:
   ```bash
   dig @127.0.0.1 srv01.lab.internal +short
   dig @127.0.0.1 -x 192.168.50.20 +short
   
   ```
### Step 2: Configure and Validate an ISC-DHCP Server
 1. Install the ISC DHCP Server package:
   ```bash
   sudo apt install -y isc-dhcp-server
   
   ```
 2. Edit /etc/dhcp/dhcpd.conf to configure an enterprise IP address pool:
   ```bash
   sudo tee /etc/dhcp/dhcpd.conf << 'EOF'
   default-lease-time 600;
   max-lease-time 7200;
   authoritative;
   
   subnet 192.168.50.0 netmask 255.255.255.0 {
     range 192.168.50.100 192.168.50.200;
     option routers 192.168.50.1;
     option domain-name-servers 192.168.50.10;
     option domain-name "lab.internal";
   
     # Static Reservation
     host printer-01 {
       hardware ethernet 00:11:22:33:44:55;
       fixed-address 192.168.50.50;
     }
   }
   EOF
   
   ```
 3. Test configuration syntax and restart the service:
   ```bash
   sudo dhcpd -t -cf /etc/dhcp/dhcpd.conf
   sudo systemctl restart isc-dhcp-server
   
   ```
 4. Inspect active leases:
   ```bash
   cat /var/lib/dhcp/dhcpd.leases
   
   ```
### Step 3: Secure and Harden the OpenSSH Service
 1. Generate SSH Key pairs on the administration host:
   ```bash
   ssh-keygen -t ed25519 -C "admin@lab.internal" -f ~/.ssh/id_ed25519 -N ""
   
   ```
 2. Install public keys into local authorized keys:
   ```bash
   cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
   chmod 600 ~/.ssh/authorized_keys
   chmod 700 ~/.ssh
   
   ```
 3. Apply hardened rules using an drop-in configuration file /etc/ssh/sshd_config.d/hardened.conf:
   ```bash
   sudo tee /etc/ssh/sshd_config.d/hardened.conf << 'EOF'
   PasswordAuthentication no
   PermitRootLogin no
   X11Forwarding no
   MaxAuthTries 3
   ClientAliveInterval 300
   ClientAliveCountMax 2
   EOF
   
   ```
 4. Test SSH configuration syntax prior to reloading:
   ```bash
   sudo sshd -t
   
   ```
 5. Reload OpenSSH daemon and verify port access:
   ```bash
   sudo systemctl reload ssh
   sudo ss -tulpn | grep sshd
   
   ```
### Step 4: Network Service Firewall Isolation with nftables
 1. Create a baseline stateful packet filter rule-set for the deployed services:
   ```bash
   sudo tee /etc/nftables.conf << 'EOF'
   #!/usr/sbin/nft -f
   
   flush ruleset
   
   table inet filter {
       chain input {
           type filter hook input priority 0; policy drop;
   
           # Loopback traffic
           iifname "lo" accept
   
           # Established and related connections
           ct state established,related accept
   
           # ICMP / Ping
           ip protocol icmp accept
   
           # Allow SSH (Port 22)
           tcp dport 22 accept
   
           # Allow DNS (Port 53 TCP/UDP)
           tcp dport 53 accept
           udp dport 53 accept
   
           # Allow DHCP Server (Port 67 UDP)
           udp dport 67 accept
       }
       chain forward {
           type filter hook forward priority 0; policy drop;
       }
       chain output {
           type filter hook output priority 0; policy accept;
       }
   }
   EOF
   
   ```
 2. Load and enable the nftables service:
   ```bash
   sudo nft -f /etc/nftables.conf
   sudo systemctl enable --now nftables
   sudo nft list ruleset
   
   ```
### Step 5: Clean Up Lab Environment
 1. Revert test configurations and restore system state:
   ```bash
   sudo systemctl stop isc-dhcp-server bind9
   sudo rm -f /etc/ssh/sshd_config.d/hardened.conf
   sudo systemctl reload ssh
   
   ```
## 12.6 Chapter Review Questions
 1. Which phase of the DHCP handshake represents the server delivering IP network options to a requesting client host?
   * A) DHCPDISCOVER
   * B) DHCPOFFER
   * C) DHCPREQUEST
   * D) DHCPACK
 2. Which BIND9 DNS resource record type maps an IP address back to its fully qualified domain name (FQDN)?
   * A) CNAME
   * B) MX
   * C) AAAA
   * D) PTR
 3. What command tests the syntax of an OpenSSH configuration file without interrupting running server sessions?
   * A) named-checkconf
   * B) sshd -t
   * C) dhcpd -t
   * D) systemctl verify sshd
 4. When implementing public-key authentication for SSH access, which file on the target host stores authorized client keys?
   * A) /etc/ssh/ssh_known_hosts
   * B) ~/.ssh/id_ed25519.pub
   * C) ~/.ssh/authorized_keys
   * D) /etc/ssh/sshd_config
## 12.7 Key Terms Glossary
 * **BIND9 (named):** The standard open-source Internet Domain Name System (DNS) software suite.
 * **DHCP DORA:** The sequence (Discover, Offer, Request, Acknowledge) used by DHCP clients and servers for IP configuration.
 * **DNS Zone File:** A text file containing mapping records between domain names, IP addresses, and operational resources.
 * **ED25519:** An elliptic-curve signature algorithm providing high security and performance for SSH key authentication.
 * **Forwarding DNS Server:** A DNS server that passes queries it cannot resolve internally to an upstream recursive DNS provider.
 * **PTR Record:** Pointer record used in reverse DNS lookups to map an IP address to a hostname.
 * **Reverse DNS:** The lookup process of converting an IP address into its corresponding domain name.
 * **SSH Public Key Authentication:** A cryptographic authentication method using key pairs (public and private) instead of passwords.
 
