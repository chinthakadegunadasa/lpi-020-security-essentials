# Chapter 11: Network Architectures and Core Protocols (Objective 024.1)
## 11.1 Enterprise Network Models and the OSI vs. TCP/IP Framework
Modern enterprise networking relies on structured layered models to standardize data communication, facilitate vendor interoperability, and streamline troubleshooting across physical and logical interfaces.
```
         OSI SEVEN-LAYER MODEL                      TCP/IP FOUR-LAYER MODEL
+------------------------------------+        +------------------------------------+
|  7. Application   (HTTP, SSH, DNS) |        |                                    |
|  6. Presentation  (TLS/SSL, ASCII) | ------>|  4. Application Layer              |
|  5. Session       (RPC, Sockets)   |        |                                    |
+------------------------------------+        +------------------------------------+
|  4. Transport     (TCP, UDP)       | ------>|  3. Transport Layer                |
+------------------------------------+        +------------------------------------+
|  3. Network       (IP, ICMP, ARP)  | ------>|  2. Internet / Network Layer       |
+------------------------------------+        +------------------------------------+
|  2. Data Link     (Ethernet, VLAN) | ------>|  1. Network Access /               |
|  1. Physical      (Fiber, Copper)  |        |     Link Layer                     |
+------------------------------------+        +------------------------------------+

```
### Protocol Encapsulation & Packet Structure
As application data flows down the protocol stack, each layer prepends a specific header containing control metadata.
 * **Data (Application Layer):** Raw payload (e.g., HTTP GET request).
 * **Segment/Datagram (Transport Layer):** Adds Source/Destination Ports, Sequence Numbers, and Checksums.
 * **Packet (Internet Layer):** Adds Source/Destination IPv4/IPv6 Addresses and TTL.
 * **Frame (Data Link Layer):** Adds Source/Destination MAC Addresses and Frame Check Sequence (FCS).
## 11.2 Core Network Protocols & Service Ports
Enterprise infrastructure relies on standard core protocols operating across specified Transport Layer ports:
| Protocol | Default Port | Transport | Description & Enterprise Use Case |
|---|---|---|---|
| **SSH** | 22 | TCP | Encrypted remote shell access and secure file transfer (SFTP/SCP). |
| **DNS** | 53 | UDP / TCP | Name resolution (UDP for queries, TCP for zone transfers and large responses). |
| **DHCP** | 67/68 | UDP | Dynamic client IP address assignment and bootstrap routing parameters. |
| **HTTP / HTTPS** | 80 / 443 | TCP | Unencrypted and TLS-encrypted web payload delivery. |
| **NTP** | 123 | UDP | High-precision clock synchronization across enterprise nodes. |
| **SNMP** | 161/162 | UDP | Network device health monitoring, metrics collection, and trap alerts. |
| **LDAPS** | 636 | TCP | Lightweight Directory Access Protocol over TLS for identity authentication. |
## 11.3 IPv4 Addressing, Subnetting, and CIDR Notation
IPv4 uses 32-bit addresses expressed in four dot-decimal octets (e.g., 192.168.10.1). Enterprise networks utilize CIDR (Classless Inter-Domain Routing) to partition networks efficiently.
```
                                  IP ADDRESS & SUBNET MASK
IP Address:   192.168.1.10  ->   11000000 . 10101000 . 00000001 . 00001010
Subnet Mask:  /24           ->   11111111 . 11111111 . 11111111 . 00000000
                                 |------------------------| |-------|
                                       Network Portion         Host Portion

```
### Common Subnet Reference Table
| Subnet Mask | CIDR Prefix | Total IPs | Usable Hosts | Enterprise Application |
|---|---|---|---|---|
| 255.255.255.0 | /24 | 256 | 254 | Standard LAN / Server Subnet |
| 255.255.255.128 | /25 | 128 | 126 | Mid-Sized Departmental Segment |
| 255.255.255.240 | /28 | 16 | 14 | DMZ / Infrastructure Management |
| 255.255.255.252 | /30 | 4 | 2 | Point-to-Point Router Interconnects |
### RFC 1918 Private IP Ranges
 * **Class A:** 10.0.0.0/8 (10.0.0.0 - 10.255.255.255)
 * **Class B:** 172.16.0.0/12 (172.16.0.0 - 172.31.255.255)
 * **Class C:** 192.168.0.0/16 (192.168.0.0 - 192.168.255.255)
## 11.4 Hands-On Lab Framework: Enterprise Network Configuration & Diagnostic Analysis
### Lab Overview
In this lab, you will configure dual-interface Linux networking, manually manage routing tables, configure persistence using modern utilities, analyze network traffic with tcpdump, and inspect protocol sockets using ss.
#### Objectives
 1. Inspect physical and virtual network interfaces using iproute2.
 2. Configure static IP addresses and custom routing table entries.
 3. Perform low-level socket inspection and diagnose listening enterprise services.
 4. Capture and filter live network traffic using tcpdump.
### Step 1: Network Interface & Address Inspection
 1. List all active network interfaces and their assigned link parameters:
   ```bash
   ip link show
   
   ```
 2. Display complete IPv4/IPv6 details for network interfaces:
   ```bash
   ip -4 address show
   
   ```
 3. Display kernel routing table information:
   ```bash
   ip route show
   
   ```
### Step 2: Configuring Static Network Interfaces and Custom Routes
 1. Create a virtual dummy interface to simulate a secondary enterprise segment:
   ```bash
   sudo ip link add dev veth_test type dummy
   sudo ip link set dev veth_test up
   
   ```
 2. Assign a static IP address (10.50.100.1/24) to the target interface:
   ```bash
   sudo ip address add 10.50.100.1/24 dev veth_test
   
   ```
 3. Add a static network route to direct traffic targeting 172.20.0.0/16 through the test interface:
   ```bash
   sudo ip route add 172.20.0.0/16 via 10.50.100.254 dev veth_test
   
   ```
 4. Verify the new static route exists in the routing table:
   ```bash
   ip route show | grep 172.20.0.0
   
   ```
### Step 3: Enterprise Socket & Listening Port Auditing
 1. Display all listening TCP and UDP sockets along with process information (-tulpn):
   ```bash
   sudo ss -tulpn
   
   ```
 2. Filter socket activity specifically for active SSH (Port 22) or DNS (Port 53) connections:
   ```bash
   sudo ss -tna 'sport = :22 or sport = :53'
   
   ```
 3. Check protocol socket statistics summary:
   ```bash
   ss -s
   
   ```
### Step 4: Network Packet Capture & Protocol Analysis
 1. Start a tcpdump session listening on the local interface capturing ICMP (ping) traffic:
   ```bash
   sudo tcpdump -i any -c 4 -nn icmp
   
   ```
 2. Open a secondary terminal tab or background session and generate ICMP traffic:
   ```bash
   ping -c 2 127.0.0.1
   
   ```
 3. Capture HTTP/HTTPS network traffic targeting a specific IP address and write to a PCAP capture file:
   ```bash
   sudo tcpdump -i any -nn -w net_capture.pcap port 80 or port 443
   
   ```
 4. Read and parse the binary capture file:
   ```bash
   tcpdump -r net_capture.pcap -nn -X | head -n 30
   
   ```
### Step 5: Clean Up Lab Environment
 1. Remove the dummy interface and associated subnets:
   ```bash
   sudo ip link delete dev veth_test
   rm -f net_capture.pcap
   
   ```
## 11.5 Chapter Review Questions
 1. Which layer of the OSI model is responsible for end-to-end data transfer, flow control, and error recovery (e.g., TCP)?
   * A) Layer 2 (Data Link)
   * B) Layer 3 (Network)
   * C) Layer 4 (Transport)
   * D) Layer 7 (Application)
 2. How many usable host IP addresses are available within a standard /28 IPv4 enterprise subnet?
   * A) 32
   * B) 30
   * C) 16
   * D) 14
 3. Which protocol operates over UDP port 123 to keep system clocks synchronized across enterprise network infrastructure?
   * A) SNMP
   * B) NTP
   * C) DHCP
   * D) LDAPS
 4. Which Linux command replaces legacy netstat for inspecting socket states, listening ports, and established TCP connections?
   * A) ip link
   * B) ifconfig
   * C) ss
   * D) route
## 11.6 Key Terms Glossary
 * **CIDR (Classless Inter-Domain Routing):** A method for allocating IP addresses and IP routing that replaces traditional classful network design.
 * **Encapsulation:** The process of wrapping protocol header and trailer metadata around data payloads as they move down network layers.
 * **IP (Internet Protocol):** The principal communications protocol in the Internet protocol suite for relaying datagrams across network boundaries.
 * **TCP (Transmission Control Protocol):** A connection-oriented, reliable transport protocol that guarantees ordered packet delivery.
 * **UDP (User Datagram Protocol):** A connectionless, lightweight transport protocol prioritized for low latency rather than reliability.
 * **VLAN (Virtual Local Area Network):** A logical subnetwork that groups collections of devices from different physical LANs into distinct broadcast domains.
 * **Subnet Mask:** A 32-bit mask used to divide an IP address into network address and host address components.
 
