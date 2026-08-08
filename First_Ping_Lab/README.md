# 📡 First Ping Lab — IP Configuration, ARP & ICMP Analysis

This directory contains the GNS3 project files and comprehensive documentation for **Lab 1: First Ping Lab**. This lab demonstrates baseline router interface configuration, local Ethernet broadcast communication, ARP resolution, ICMP Ping reachability, and basic packet troubleshooting.

---

## 📋 Table of Contents
1. [Router & Host Configurations](#1-router--host-configurations)
2. [Ping & ICMP Overview](#2-ping--icmp-overview)
3. [Ping Errors & Packet Loss Troubleshooting](#3-ping-errors--packet-loss-troubleshooting)
4. [ARP & ICMP Step-by-Step Communication Flow](#4-arp--icmp-step-by-step-communication-flow)
5. [ARP Table vs. MAC Address Table](#5-arp-table-vs-mac-address-table)
6. [Technical Notes & Protocol Clarifications](#6-technical-notes--protocol-clarifications)

---

## 1. Router & Host Configurations

### 🔹 Router 1 (R1) Configuration
```text
R1 > enable
R1 # config terminal
R1(config)# interface g1/0
R1(config-if)# ip add 192.168.12.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit
```

**Verification:**
```text
R1# show ip interface brief

Interface              IP-Address      OK? Method Status                Protocol
FastEthernet0/0        unassigned      YES unset  administratively down down
GigabitEthernet1/0     192.168.12.1    YES manual up                    up
```

---

### 🔹 Router 2 (R2) Configuration
```text
R2 > enable
R2 # config terminal
R2(config)# interface g1/0
R2(config-if)# ip add 192.168.12.2 255.255.255.0
R2(config-if)# no shutdown
R2(config-if)# exit
```

**Verification:**
```text
R2# show ip interface brief

Interface              IP-Address      OK? Method Status                Protocol
FastEthernet0/0        unassigned      YES unset  administratively down down
GigabitEthernet1/0     192.168.12.2    YES manual up                    up
```

---

### 🔹 Router 3 (R3) & Subnet Gateway Configuration

* **R3 GigabitEthernet1/0**: `192.168.1.100 /24`
* **R3 GigabitEthernet2/0**: `192.168.2.100 /24`

#### PC Configurations:
* **PC3**:
  - IP Address: `192.168.1.1 /24`
  - Default Gateway: `192.168.1.100`
* **PC4**:
  - IP Address: `192.168.2.1 /24`
  - Default Gateway: `192.168.2.100`

---

## 2. Ping & ICMP Overview

### What is Ping?
**Ping** is a diagnostic utility used to test reachability between host devices across an IP network. It measures:
- **Connectivity**: Whether the remote destination is reachable.
- **Latency / Round-Trip Time (RTT)**: Time taken for packets to travel to the destination and back.

### Protocol Details
- **Protocol**: **ICMP** (Internet Control Message Protocol)
- **OSI Layer**: Layer 3 (Network Layer) — encapsulation directly within IP packets (IP Protocol Number 1).
- **Core Messages**:
  - `ICMP Type 8`: Echo Request
  - `ICMP Type 0`: Echo Reply

---

## 3. Ping Errors & Packet Loss Troubleshooting

| Symptom / Error | Common Causes | Solution / Action |
| :--- | :--- | :--- |
| **Request Timed Out** | Target device is down, host firewall is dropping ICMP, or severe network congestion. | Verify physical/link status, check target host firewall rules. |
| **Destination Host Unreachable** | No route to destination, gateway missing, or remote network disconnected. | Check IP configuration, default gateway settings, and routing tables (`show ip route`). |
| **High Packet Loss (e.g. 75%+ loss)** | Network congestion, faulty cables/hardware, high CPU load, or wireless degradation. | Inspect hardware logs, test interfaces, check bandwidth utilization. |

---

## 4. ARP & ICMP Step-by-Step Communication Flow

Consider two hosts connected on a local switch:
- **PC1**: IP `192.168.1.1/24`, MAC `AA`
- **PC2**: IP `192.168.1.2/24`, MAC `BB`

```text
PC1 (192.168.1.1 / MAC: AA) <------ Switch1 ------> PC2 (192.168.1.2 / MAC: BB)
```

### Step 1: PC1 Prepares ICMP Echo Request
PC1 needs to ping PC2 (`192.168.1.2`).
- Source IP: `192.168.1.1` | Destination IP: `192.168.1.2`
- Source MAC: `AA` | Destination MAC: **Unknown (`?`)**

Since the destination MAC is unknown, PC1 holds the ICMP packet in memory and initiates an **ARP Request**.

---

### Step 2: Broadcast ARP Request
PC1 broadcasts an ARP frame to find PC2's MAC address:
- **Sender MAC**: `AA` | **Sender IP**: `192.168.1.1`
- **Target MAC**: `FF:FF:FF:FF:FF:FF` (Broadcast) | **Target IP**: `192.168.1.2`

*Switch1 receives the frame on port `e0`, learns that MAC `AA` is on `e0`, and floods the broadcast frame out all other ports (e.g., `e1`).*

---

### Step 3: Unicast ARP Reply
PC2 receives the ARP request, sees its own IP (`192.168.1.2`), and replies directly to PC1:
- **Sender MAC**: `BB` | **Sender IP**: `192.168.1.2`
- **Target MAC**: `AA` | **Target IP**: `192.168.1.1`

*Switch1 receives the reply on port `e1`, learns that MAC `BB` is on `e1`, and forwards the unicast frame to `e0` (PC1).*

---

### Step 4: Updating Tables & Sending ICMP Packets
1. **PC1 ARP Table**: Stores mapping `192.168.1.2 -> BB`.
2. **Switch MAC Table**: Learns `AA` on `e0`, `BB` on `e1`.

**ICMP Echo Request (PC1 → PC2):**
- Source MAC: `AA` | Destination MAC: `BB`
- Source IP: `192.168.1.1` | Destination IP: `192.168.1.2`

**ICMP Echo Reply (PC2 → PC1):**
- Source MAC: `BB` | Destination MAC: `AA`
- Source IP: `192.168.1.2` | Destination IP: `192.168.1.1`

---

## 5. ARP Table vs. MAC Address Table

```text
+-----------------------------------------------------------------------+
|                              ARP Table                                |
| (Maintained by End Hosts & Routers at Layer 3)                        |
|                                                                       |
| Maps IP Address --------------> MAC Address                           |
| Example: 192.168.1.2 ---------> BB:BB:BB:BB:BB:BB                      |
+-----------------------------------------------------------------------+

+-----------------------------------------------------------------------+
|                          MAC Address Table                            |
| (Maintained by Switches at Layer 2)                                   |
|                                                                       |
| Maps Switch Port -------------> MAC Address                           |
| Example: Port Ethernet0 --------> AA:AA:AA:AA:AA:AA                   |
|          Port Ethernet1 --------> BB:BB:BB:BB:BB:BB                   |
+-----------------------------------------------------------------------+
```

---

## 6. Technical Notes & Protocol Clarifications

> [!NOTE]  
> **Protocol Clarification**:
> - **ICMP (Internet Control Message Protocol)**: Operates at Layer 3 directly over IP (IP Protocol 1). Ping utilities use ICMP Echo Request (Type 8) and Echo Reply (Type 0). ICMP does not use TCP/UDP port numbers.
> - **IGMP (Internet Group Management Protocol)**: Operates at Layer 3 (IP Protocol 2) and is used by IPv4 hosts to report their multicast group memberships to neighboring multicast routers.
> - **ARP (Address Resolution Protocol)**: Operates between Layer 2 and Layer 3 to map known 32-bit IPv4 addresses to 48-bit Ethernet MAC addresses within a local broadcast domain.
