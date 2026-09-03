# 🔗 EtherChannel Configuration — LACP & PAgP

A Cisco switching lab demonstrating **EtherChannel (Link Aggregation)** using two different negotiation protocols:

- **LACP (Link Aggregation Control Protocol)**
- **PAgP (Port Aggregation Protocol)**

The lab is designed using Cisco Catalyst 2960 switches in Cisco Packet Tracer and demonstrates how multiple physical Ethernet links can be combined into a single logical EtherChannel interface.

---

## 📌 Lab Overview

EtherChannel combines multiple physical interfaces into one logical interface called a **Port-Channel**.

Instead of treating several links independently, EtherChannel allows the switch to:

- Increase aggregate bandwidth
- Provide link redundancy
- Reduce the impact of a single physical-link failure
- Simplify management by operating multiple links as one logical interface
- Prevent Layer 2 STP from blocking individual links within the EtherChannel

This lab covers two EtherChannel negotiation protocols:

| Protocol | Standard | Vendor Support | Example Mode |
|----------|----------|----------------|--------------|
| **LACP** | IEEE 802.3ad / 802.1AX | Multi-vendor | `active` |
| **PAgP** | Cisco Proprietary | Cisco devices | `desirable` |

---

## 🖥️ Topology

```text
                    EtherChannel
        =====================================
        ||                                  ||
        ||                                  ||
   +-----------+                       +-----------+
   |  Switch0  |                       |  Switch1  |
   | 2960-24TT |                       | 2960-24TT |
   +-----------+                       +-----------+
        ||                                  ||
        ||                                  ||
     Fa0/1                                Fa0/1
     Fa0/2                                Fa0/2
        \                                  /
         \                                /
          ======= Port-Channel ==========
```

The lab can be implemented as separate EtherChannel scenarios:

### Scenario 1 — LACP

```text
Switch0                         Switch1
---------                       ---------
Fa0/1 ======================== Fa0/1
Fa0/2 ======================== Fa0/2

        Port-Channel 1
        LACP
```

### Scenario 2 — PAgP

```text
Switch0                         Switch1
---------                       ---------
Fa0/1 ======================== Fa0/1
Fa0/2 ======================== Fa0/2
Fa0/3 ======================== Fa0/3

        Port-Channel 1
        PAgP
```

---

# 🧠 EtherChannel Concepts

## What is EtherChannel?

EtherChannel is a Cisco technology that logically combines multiple physical Ethernet links into a single logical link.

For example:

```text
Fa0/1 ───────┐
Fa0/2 ───────┼──── Port-Channel 1
Fa0/3 ───────┘
```

The switch treats the physical interfaces as members of the same logical bundle.

---

# 🔵 LACP

**LACP (Link Aggregation Control Protocol)** is an open standard used to negotiate EtherChannel formation.

LACP is supported by Cisco and many other network vendors.

### LACP Modes

| Mode      | Description                             |
| --------- | --------------------------------------- |
| `active`  | Actively sends LACP negotiation packets |
| `passive` | Waits for LACP negotiation packets      |

A channel can form when:

```text
active + active
```

or

```text
active + passive
```

A channel will **not** form with:

```text
passive + passive
```

---

# 🟢 PAgP

**PAgP (Port Aggregation Protocol)** is a Cisco proprietary EtherChannel negotiation protocol.

### PAgP Modes

| Mode        | Description                     |
| ----------- | ------------------------------- |
| `desirable` | Actively attempts to negotiate  |
| `auto`      | Passively waits for negotiation |

A channel can form using:

```text
desirable + desirable
```

or

```text
desirable + auto
```

A channel will **not** form using:

```text
auto + auto
```

---

# ⚙️ Configuration

## 1. LACP Configuration

### Switch0

```cisco
enable
configure terminal

interface range fa0/1 - 2
channel-group 1 mode active
exit

interface port-channel 1
exit

end
write memory
```

### Switch1

```cisco
enable
configure terminal

interface range fa0/1 - 2
channel-group 1 mode active
exit

interface port-channel 1
exit

end
write memory
```

This creates:

```text
Port-Channel 1
├── Fa0/1
└── Fa0/2
```

using LACP.

---

# ⚙️ 2. PAgP Configuration

### Switch0

```cisco
enable
configure terminal

interface range fa0/1 - 3
channel-group 1 mode desirable
exit

interface port-channel 1
exit

end
write memory
```

### Switch1

```cisco
enable
configure terminal

interface range fa0/1 - 3
channel-group 1 mode desirable
exit

interface port-channel 1
exit

end
write memory
```

This creates:

```text
Port-Channel 1
├── Fa0/1
├── Fa0/2
└── Fa0/3
```

using PAgP.

---

# 🔍 Verification Commands

After configuring EtherChannel, verify the configuration using the following commands.

## Check EtherChannel Summary

```cisco
show etherchannel summary
```

Example:

```text
Group  Port-channel  Protocol    Ports
------+-------------+-----------+--------------------------
1      Po1(SU)       LACP        Fa0/1(P) Fa0/2(P)
```

Important indicators:

```text
Po1(SU)
```

Where:

* `S` = Layer 2 EtherChannel
* `U` = Port-Channel is in use

Member interfaces should normally show:

```text
(P)
```

which indicates that the port is successfully bundled into the EtherChannel.

---

## Check Port-Channel Interfaces

```cisco
show interfaces port-channel 1
```

---

## Check LACP Neighbors

```cisco
show lacp neighbor
```

This is particularly useful when troubleshooting LACP.

---

## Check PAgP Neighbor Information

```cisco
show pagp neighbor
```

This is useful when troubleshooting PAgP.

---

## Check Individual Interfaces

```cisco
show interfaces fa0/1
show interfaces fa0/2
show interfaces fa0/3
```

---

## Check Running Configuration

```cisco
show running-config
```

---

# 🧪 Testing the EtherChannel

After the EtherChannel is established, verify connectivity between the switches.

For example:

```cisco
ping <destination-ip>
```

You can also shut down one physical member link to verify redundancy.

Example:

```cisco
configure terminal
interface fa0/1
shutdown
end
```

Then verify:

```cisco
show etherchannel summary
```

The remaining member links should continue forwarding traffic as long as the EtherChannel still has active members.

Re-enable the interface:

```cisco
configure terminal
interface fa0/1
no shutdown
end
```

---

# ⚠️ Important EtherChannel Requirements

All physical interfaces participating in the same EtherChannel should have matching configurations.

Important parameters include:

* Speed
* Duplex
* VLAN configuration
* Trunk/access mode
* Native VLAN
* Allowed VLANs
* Channel-group configuration
* Negotiation protocol

For example, if the EtherChannel is configured as a trunk, configure the trunk consistently across the member interfaces.

---

# 🛠️ Troubleshooting

## EtherChannel Does Not Form

Check:

```cisco
show etherchannel summary
```

Then verify:

```cisco
show interfaces status
```

and:

```cisco
show running-config
```

### Common causes

#### 1. Incorrect negotiation modes

For LACP:

```text
active + active      ✅
active + passive     ✅
passive + passive    ❌
```

For PAgP:

```text
desirable + desirable   ✅
desirable + auto        ✅
auto + auto             ❌
```

---

#### 2. Mismatched interface configuration

Make sure both sides have compatible:

```text
Speed
Duplex
VLAN
Trunk configuration
Native VLAN
Allowed VLANs
```

---

#### 3. Wrong protocol

Do not configure one side with LACP and the other side with PAgP.

Incorrect:

```text
Switch0 → LACP
Switch1 → PAgP
```

Correct:

```text
Switch0 → LACP
Switch1 → LACP
```

or:

```text
Switch0 → PAgP
Switch1 → PAgP
```

---

#### 4. Physical connectivity problem

Check:

```cisco
show interfaces status
```

Make sure the member interfaces are physically connected and operational.

---

# 📊 LACP vs PAgP

| Feature                         | LACP                              | PAgP                      |
| ------------------------------- | --------------------------------- | ------------------------- |
| Full Name                       | Link Aggregation Control Protocol | Port Aggregation Protocol |
| Standard                        | IEEE                              | Cisco Proprietary         |
| Multi-vendor                    | ✅ Yes                             | ❌ No                      |
| Cisco Support                   | ✅ Yes                             | ✅ Yes                     |
| Active Mode                     | `active`                          | `desirable`               |
| Passive Mode                    | `passive`                         | `auto`                    |
| Recommended for modern networks | ✅                                 | Less commonly used        |
| EtherChannel Support            | ✅                                 | ✅                         |

---

# 📁 Suggested Repository Structure

```text
EtherChannel-LACP-PAgP/
│
├── README.md
│
├── topology/
│   └── etherchannel-topology.png
│
├── configs/
│   ├── LACP/
│   │   ├── Switch0.txt
│   │   └── Switch1.txt
│   │
│   └── PAgP/
│       ├── Switch0.txt
│       └── Switch1.txt
│
└── packet-tracer/
    └── EtherChannel-LACP-PAgP.pkt
```

---

# 🎯 Learning Objectives

By completing this lab, you will understand:

* What EtherChannel is
* Why link aggregation is used
* How to configure EtherChannel
* How LACP works
* How PAgP works
* Difference between LACP and PAgP
* LACP `active` and `passive` modes
* PAgP `desirable` and `auto` modes
* How to verify EtherChannel status
* How to troubleshoot failed EtherChannel formation
* How EtherChannel provides redundancy and aggregated bandwidth

---

# 🏁 Conclusion

This lab demonstrates how multiple physical Ethernet links can be combined into a single logical EtherChannel using **LACP** and **PAgP**.

LACP is generally preferred for modern deployments because it is an open standard and supports interoperability between different vendors. PAgP remains useful when working in Cisco-only environments and when learning legacy Cisco EtherChannel technologies.

---

## 👨‍💻 Author

**Chakresh Ram**

Networking & Cybersecurity Lab Portfolio

---

## ⭐ Topics

```text
Cisco
Cisco Packet Tracer
EtherChannel
Link Aggregation
LACP
PAgP
Port-Channel
Switching
Network Redundancy
Network Engineering
CCNA
```
