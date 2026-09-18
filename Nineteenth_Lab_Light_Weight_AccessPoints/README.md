# Cisco Lightweight AP & WLC Lab

![Cisco](https://img.shields.io/badge/Cisco-Packet%20Tracer-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![Networking](https://img.shields.io/badge/Networking-Wireless-blue?style=for-the-badge)
![WLC](https://img.shields.io/badge/Wireless-WLC%20%2B%20LAP-green?style=for-the-badge)
![DHCP](https://img.shields.io/badge/DHCP-Configured-orange?style=for-the-badge)

A Cisco Packet Tracer laboratory demonstrating the deployment of a **Lightweight Access Point (LAP)** managed by a **Wireless LAN Controller (WLC)**.

The lab covers wireless LAN controller management, Lightweight AP registration, WLAN/SSID configuration, WPA2-PSK security, DHCP-based client addressing, and wireless client connectivity.

---

## 📌 Project Overview

This lab simulates a small enterprise wireless network where the **Wireless LAN Controller (WLC)** centrally manages a **Lightweight Access Point (LAP)**.

The router provides:

- Default gateway
- DHCP services
- DNS information

The WLC provides:

- Wireless LAN management
- WLAN/SSID configuration
- Wireless security
- Lightweight AP management

Wireless clients connect to the LAP using the configured SSID and receive their IP configuration through DHCP.

---

## 🏗️ Network Topology

```text
                         ┌──────────────────┐
                         │   Cisco ISR4321  │
                         │     Router       │
                         │                  │
                         │ G0/0/0           │
                         │ 192.168.1.1/24   │
                         └────────┬─────────┘
                                  │
                                  │
                         ┌────────▼─────────┐
                         │   Cisco 2960     │
                         │     Switch       │
                         └───┬────┬─────┬───┘
                             │    │     │
                             │    │     │
                    ┌────────▼┐   │   ┌─▼──────────────┐
                    │   WLC   │   │   │ Lightweight AP│
                    │192.168.1.2│ │   │  192.168.1.5  │
                    └─────────┘   │   └───────┬───────┘
                                  │           │
                                  │        Wireless
                             ┌────▼────┐  ┌───▼────────┐
                             │   PC0   │  │  Clients   │
                             │  DHCP   │  │            │
                             └─────────┘  │ Laptop     │
                                          │ Smartphone │
                                          └────────────┘
```

---

## 🔧 Devices Used

| Device | Role |
|---|---|
| Cisco ISR4321 | Router / DHCP Server |
| Cisco 2960-24TT | Layer 2 Switch |
| Cisco WLC-3504 | Wireless LAN Controller |
| LAP-PT | Lightweight Access Point |
| PC-PT | Wired Management/Test Client |
| Laptop-PT | Wireless Client |
| Smartphone-PT | Wireless Client |

---

## 🌐 IP Addressing

| Device | Interface | IP Address | Subnet Mask | Gateway |
|---|---|---|---|---|
| Router | GigabitEthernet0/0/0 | `192.168.1.1` | `255.255.255.0` | — |
| WLC | Management | `192.168.1.2` | `255.255.255.0` | `192.168.1.1` |
| Lightweight AP | GigabitEthernet0 | `192.168.1.5` | `/24` | `192.168.1.1` |
| PC0 | Fa0 | DHCP | `/24` | DHCP |
| Laptop0 | Wireless0 | DHCP | `/24` | DHCP |
| Smartphone0 | Wireless0 | DHCP | `/24` | DHCP |

---

## 📡 Wireless Configuration

| Parameter | Configuration |
|---|---|
| SSID | `chakresh` |
| Security | WPA2-PSK |
| Encryption | AES |
| PSK Passphrase | `chakresh` |
| IP Assignment | DHCP |

> **Security note:** The PSK above is used only for this Packet Tracer lab. Do not reuse lab credentials in real networks.

---

# ⚙️ Configuration

## 1. Router Configuration

The router is configured as the default gateway and DHCP server.

```cisco
enable
configure terminal

interface gigabitEthernet0/0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
exit

ip dhcp pool chakresh
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
 dns-server 8.8.8.8
exit

ip dhcp excluded-address 192.168.1.1 192.168.1.4

end
write memory
```

The DHCP exclusion reserves:

```text
192.168.1.1
192.168.1.2
192.168.1.3
192.168.1.4
```

for infrastructure devices.

---

# 📶 2. WLC Configuration

The WLC management interface is configured with:

```text
IP Address:       192.168.1.2
Subnet Mask:      255.255.255.0
Default Gateway:  192.168.1.1
DNS Server:       8.8.8.8
```

The WLC web management interface can be accessed from PC0:

```text
https://192.168.1.2
```

---

# 📡 3. WLAN / SSID Configuration

A WLAN was created on the WLC with:

```text
WLAN ID:      1
Profile Name: chakresh
SSID:         chakresh
```

Wireless security:

```text
Security:    WPA2-PSK
Encryption:  AES
Passphrase:  chakresh
```

The WLAN is enabled and provided through the Lightweight AP.

---

# 📡 4. Lightweight AP Configuration

The Lightweight AP is connected to the switch and registers with the WLC using CAPWAP.

AP addressing:

```text
IP Address: 192.168.1.5/24
Gateway:    192.168.1.1
```

The AP should display a CAPWAP relationship with the controller:

```text
CAPWAP Status:
Connected to 192.168.1.2
```

The AP then provides the WLAN configured on the WLC.

---

# 💻 5. Wireless Client Configuration

The laptop and smartphone use DHCP for automatic network configuration.

Example laptop configuration:

```text
SSID:           chakresh
Authentication: WPA2-PSK
Encryption:     AES
IP Assignment:  DHCP
```

The laptop received:

```text
IPv4 Address: 192.168.1.7
Subnet Mask:  255.255.255.0
```

---

# 🔄 Network Communication Flow

```text
Wireless Client
      │
      │  Associate with SSID
      ▼
Lightweight AP
      │
      │  CAPWAP
      ▼
     WLC
      │
      ▼
   Switch
      │
      ▼
   Router
      │
      │ DHCP
      ▼
Wireless Client
```

The WLC centrally manages the wireless WLAN while the LAP provides wireless access to clients.

---

# 🧪 Verification

## Verify Router Interface

```cisco
show ip interface brief
```

Expected:

```text
GigabitEthernet0/0/0    192.168.1.1    up    up
```

## Verify DHCP

```cisco
show ip dhcp pool
show ip dhcp binding
```

## Verify WLC Management

Open from PC0:

```text
https://192.168.1.2
```

The WLC dashboard should display the controller information and management IP.

## Verify WLAN

The WLC WLAN page should show:

```text
WLAN ID:      1
Profile Name: chakresh
WLAN SSID:    chakresh
```

## Verify Lightweight AP

The AP should show:

```text
CAPWAP Status: Connected
Controller:    192.168.1.2
IP Address:    192.168.1.5
```

## Verify Wireless Client

The wireless client should receive an address from the router's DHCP pool.

Example:

```text
IP Address:    192.168.1.7
Subnet Mask:   255.255.255.0
Gateway:       192.168.1.1
```

---

# 🛠️ Troubleshooting

During testing, the router displayed:

```text
%IP-4-DUPADDR: Duplicate address 192.168.1.1
```

This indicates that another device or interface was using `192.168.1.1`.

The intended addressing scheme is:

```text
Router → 192.168.1.1
WLC    → 192.168.1.2
LAP    → 192.168.1.5
```

Each infrastructure device must have a unique IP address.

Useful verification commands:

```cisco
show ip interface brief
show arp
show running-config
```

> **Important:** Resolve the duplicate IP warning before treating the topology as fully validated.

---

# 📚 Concepts Demonstrated

- Cisco Wireless LAN Controller
- Lightweight Access Point
- CAPWAP
- Centralized wireless management
- WLAN / SSID configuration
- WPA2-PSK
- AES encryption
- DHCP
- Default gateway
- DNS configuration
- Wireless client association
- Layer 2 switching
- IP addressing
- Wireless network troubleshooting
- Network verification

---

# 🎯 Learning Objectives

After completing this lab, you should understand how to:

1. Configure a Cisco router as a DHCP server.
2. Configure a WLC management interface.
3. Create a WLAN/SSID on a WLC.
4. Configure WPA2-PSK wireless security.
5. Connect a Lightweight AP to a WLC.
6. Understand the purpose of CAPWAP.
7. Connect wireless clients to an enterprise WLAN.
8. Verify DHCP address allocation.
9. Troubleshoot IP addressing conflicts.
10. Validate wireless network connectivity.

---

# 📁 Recommended Repository Structure

```text
Cisco-Lightweight-AP-WLC-Lab/
│
├── README.md
│
├── packet-tracer/
│   └── Lightweight-AP-WLC-Lab.pkt
│
├── screenshots/
│   ├── topology.png
│   ├── router-dhcp.png
│   ├── wlc-management.png
│   ├── wlc-dashboard.png
│   ├── wlan.png
│   ├── lightweight-ap.png
│   └── wireless-client.png
│
└── docs/
    └── configuration.txt
```

---

# 🔐 Lab Scope

This project was created for **educational and network administration practice** using Cisco Packet Tracer.

The environment is simulated and isolated. No production infrastructure or external network is targeted.

---

# 👨‍💻 Author

**Chakresh Ram Kudupudi**

Cyber Security | Networking | SOC | VAPT

---

## ⭐ Key Takeaway

This laboratory demonstrates a basic enterprise wireless architecture:

```text
                ┌─────────────┐
                │    Router   │
                │ DHCP/Gateway│
                └──────┬──────┘
                       │
                ┌──────▼──────┐
                │   Switch    │
                └───┬─────┬───┘
                    │     │
              ┌─────▼─┐ ┌─▼─────┐
              │  WLC  │ │  PC   │
              └───┬───┘ └───────┘
                  │
                CAPWAP
                  │
              ┌───▼────┐
              │  LAP   │
              └───┬────┘
                  │
             ┌────▼─────┐
             │ Wireless │
             │ Clients  │
             └──────────┘
```

**Router → DHCP/Gateway | WLC → Wireless Control | LAP → Wireless Access | Clients → WLAN**
