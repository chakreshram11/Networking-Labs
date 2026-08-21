# 🌐 Lab 7: Multi-Area OSPF & eBGP Redistribution Lab

Welcome to **Lab 7: Multi-Area OSPF & eBGP Redistribution**. This GNS3 lab simulates an enterprise network with a Multi-Area OSPF core (Area 0, Area 1, Area 2), two Area Border Routers (ABRs), an Autonomous System Border Router (ASBR) with eBGP peering to an external ISP network (AS 200), mutual route redistribution, and an integrated DHCP server.

---

## 📐 Network Topology & Architecture

![Topology Screenshot](images/topology.png)

### 🔹 Autonomous Systems & OSPF Areas
- **OSPF Domain (Process ID 110)**:
  - **Area 0 (Backbone)**: `R1`, `R2`, `R3` (Interconnects `192.168.1.0/24` and `192.168.2.0/24`)
  - **Area 1**: `R2`, `R4`, `R5` (Subnets `192.168.3.0/24`, `192.168.5.0/24`, `192.168.7.0/24`)
  - **Area 2**: `R3`, `R6`, `R7` (Subnets `192.168.4.0/24`, `192.168.6.0/24`)
- **BGP Domain**:
  - **AS 100**: Router `R1` (Local ASBR)
  - **AS 200**: Router `R8` (External ISP / Remote AS)
  - **eBGP Peering Link**: `192.168.20.0/24` between `R1` (`192.168.20.1`) and `R8` (`192.168.20.2`)

### 🔹 Key Device Roles
| Device | Role | IP / Interfaces | Routing Protocols |
|---|---|---|---|
| **R1** | ASBR / Area 0 Router | `1.1.1.1/32` (Lo0), `192.168.1.1` (Fa0/0), `192.168.2.1` (Fa0/1), `192.168.20.1` (Fa1/0) | OSPF 110 (Area 0), BGP 100 (eBGP to R8), Mutual Redistribution |
| **R2** | ABR (Area 0 ↔ Area 1) | `2.2.2.2/32` (Lo0), `192.168.1.2` (Fa0/0), `192.168.3.1` (Fa0/1) | OSPF 110 (Area 0, Area 1) |
| **R3** | ABR (Area 0 ↔ Area 2) | `3.3.3.3/32` (Lo0), `192.168.2.2` (Fa0/0), `192.168.4.1` (Fa0/1) | OSPF 110 (Area 0, Area 2) |
| **R4** | Area 1 Internal Router | `4.4.4.4/32` (Lo0), `192.168.3.2` (Fa0/0), `192.168.5.1` (Fa0/1) | OSPF 110 (Area 1) |
| **R5** | Area 1 Internal Router & DHCP Server | `5.5.5.5/32` (Lo0), `192.168.5.2` (Fa0/0), `192.168.7.1` (Fa0/1) | OSPF 110 (Area 1), Cisco IOS DHCP (`ospf-pool`) |
| **R6** | Area 2 Internal Router | `6.6.6.6/32` (Lo0), `192.168.4.2` (Fa0/0), `192.168.6.1` (Fa0/1) | OSPF 110 (Area 2) |
| **R7** | Area 2 Internal Router | `7.7.7.7/32` (Lo0), `192.168.6.2` (Fa0/0) | OSPF 110 (Area 2) |
| **R8** | ISP / External BGP Peer | `8.8.8.8/32` (Lo0), `192.168.20.2` (Fa0/0) | BGP 200 (eBGP to R1) |
| **PC1**| Client Host | Dynamic DHCP (`192.168.7.0/24`) | Default Gateway `192.168.7.1` (R5) |

---

## ⚙️ Device Configurations & CLI Transcripts

### 🖥️ Router R1 (ASBR - Area 0 & BGP AS 100)

```text
R1#config t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#int g1/0
R1(config-if)#ip add 192.168.1.1 255.255.255.0
R1(config-if)#no shut
*Aug 21 12:19:46.683: %LINK-3-UPDOWN: Interface GigabitEthernet1/0, changed state to up
*Aug 21 12:19:47.683: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet1/0, changed state to up
R1(config-if)#int g2/0
R1(config-if)#ip add 192.168.2.1 255.255.255.0
R1(config-if)#no shut
*Aug 21 12:20:25.267: %LINK-3-UPDOWN: Interface GigabitEthernet2/0, changed state to up
*Aug 21 12:20:26.267: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet2/0, changed state to up
R1(config-if)#int g3/0
R1(config-if)#ip add 192.168.20.1 255.255.255.0
R1(config-if)#no shut
*Aug 21 12:20:59.227: %LINK-3-UPDOWN: Interface GigabitEthernet3/0, changed state to up
*Aug 21 12:21:00.227: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet3/0, changed state to up
R1(config-if)#int lo0
R1(config-if)#ip add 1.1.1.1 255.255.255.255
*Aug 21 12:21:24.323: %LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback0, changed state to up
R1(config-if)#exit

R1(config)#router ospf 110
R1(config-router)#network 192.168.1.0 0.0.0.255 area 0
R1(config-router)#network 192.168.2.0 0.0.0.255 area 0
R1(config-router)#network 1.1.1.1 0.0.0.0 area 0
*Aug 21 12:43:03.131: %OSPF-5-ADJCHG: Process 110, Nbr 2.2.2.2 on GigabitEthernet1/0 from LOADING to FULL, Loading Done
*Aug 21 12:45:13.983: %OSPF-5-ADJCHG: Process 110, Nbr 3.3.3.3 on GigabitEthernet2/0 from LOADING to FULL, Loading Done
R1(config-router)#exit

R1(config)#router bgp 100
R1(config-router)#neighbor 192.168.20.2 remote-as 200
R1(config-router)#redistribute ospf 110
R1(config-router)#exit

R1(config)#router ospf 110
R1(config-router)#redistribute bgp 100 subnets
R1(config-router)#do write
Building configuration...
[OK]
*Aug 21 13:00:36.199: %BGP-5-ADJCHANGE: neighbor 192.168.20.2 Up
R1(config-router)#do ping 8.8.8.8
Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 20/29/36 ms
```

---

### 🖥️ Router R2 (ABR - Area 0 & Area 1)

```text
R2#config t
Enter configuration commands, one per line.  End with CNTL/Z.
R2(config)#int g1/0
R2(config-if)#ip add 192.168.1.2 255.255.255.0
R2(config-if)#no shut
*Aug 21 12:22:20.751: %LINK-3-UPDOWN: Interface GigabitEthernet1/0, changed state to up
*Aug 21 12:22:21.751: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet1/0, changed state to up
R2(config-if)#int g2/0
R2(config-if)#ip add 192.168.3.1 255.255.255.0
R2(config-if)#no shut
*Aug 21 12:22:43.879: %LINK-3-UPDOWN: Interface GigabitEthernet2/0, changed state to up
*Aug 21 12:22:44.879: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet2/0, changed state to up
R2(config-if)#int lo0
R2(config-if)#ip add 2.2.2.2 255.255.255.255
*Aug 21 12:22:51.167: %LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback0, changed state to up
R2(config-if)#exit

R2(config)#router ospf 110
R2(config-router)#network 2.2.2.2 0.0.0.0 area 0
R2(config-router)#network 192.168.1.0 0.0.0.255 area 0
R2(config-router)#network 192.168.3.0 0.0.0.255 area 1
*Aug 21 12:43:03.095: %OSPF-5-ADJCHG: Process 110, Nbr 1.1.1.1 on GigabitEthernet1/0 from LOADING to FULL, Loading Done
*Aug 21 12:46:59.531: %OSPF-5-ADJCHG: Process 110, Nbr 4.4.4.4 on GigabitEthernet2/0 from LOADING to FULL, Loading Done
R2(config-router)#do write
Building configuration...
[OK]
```

---

### 🖥️ Router R3 (ABR - Area 0 & Area 2)

```text
R3#config t
Enter configuration commands, one per line.  End with CNTL/Z.
R3(config)#int g1/0
R3(config-if)#ip add 192.168.2.2 255.255.255.0
R3(config-if)#no shut
*Aug 21 12:24:36.827: %LINK-3-UPDOWN: Interface GigabitEthernet1/0, changed state to up
*Aug 21 12:24:37.827: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet1/0, changed state to up
R3(config-if)#int g2/0
R3(config-if)#ip add 192.168.4.1 255.255.255.0
R3(config-if)#no shut
*Aug 21 12:25:27.351: %LINK-3-UPDOWN: Interface GigabitEthernet2/0, changed state to up
*Aug 21 12:25:28.351: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet2/0, changed state to up
R3(config-if)#int lo0
R3(config-if)#ip add 3.3.3.3 255.255.255.255
*Aug 21 12:25:30.563: %LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback0, changed state to up
R3(config-if)#exit

R3(config)#router ospf 110
R3(config-router)#network 3.3.3.3 0.0.0.0 area 0
R3(config-router)#network 192.168.2.0 0.0.0.255 area 0
R3(config-router)#network 192.168.4.0 0.0.0.255 area 2
*Aug 21 12:45:13.879: %OSPF-5-ADJCHG: Process 110, Nbr 1.1.1.1 on GigabitEthernet1/0 from LOADING to FULL, Loading Done
*Aug 21 12:49:58.627: %OSPF-5-ADJCHG: Process 110, Nbr 6.6.6.6 on GigabitEthernet2/0 from LOADING to FULL, Loading Done
R3(config-router)#do write
Building configuration...
[OK]
```

---

### 🖥️ Router R4 (Area 1 Internal Router)

```text
R4#config t
Enter configuration commands, one per line.  End with CNTL/Z.
R4(config)#int g1/0
R4(config-if)#ip add 192.168.3.2 255.255.255.0
R4(config-if)#no shut
*Aug 21 12:27:13.287: %LINK-3-UPDOWN: Interface GigabitEthernet1/0, changed state to up
*Aug 21 12:27:14.287: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet1/0, changed state to up
R4(config-if)#int g2/0
R4(config-if)#ip add 192.168.5.1 255.255.255.0
R4(config-if)#no shut
*Aug 21 12:27:33.931: %LINK-3-UPDOWN: Interface GigabitEthernet2/0, changed state to up
*Aug 21 12:27:34.931: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet2/0, changed state to up
R4(config-if)#int lo0
R4(config-if)#ip add 4.4.4.4 255.255.255.255
*Aug 21 12:27:36.007: %LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback0, changed state to up
R4(config-if)#exit

R4(config)#router ospf 110
R4(config-router)#network 4.4.4.4 0.0.0.0 area 1
R4(config-router)#network 192.168.3.0 0.0.0.255 area 1
R4(config-router)#network 192.168.5.0 0.0.0.255 area 1
*Aug 21 12:46:59.035: %OSPF-5-ADJCHG: Process 110, Nbr 2.2.2.2 on GigabitEthernet1/0 from LOADING to FULL, Loading Done
*Aug 21 12:48:49.859: %OSPF-5-ADJCHG: Process 110, Nbr 5.5.5.5 on GigabitEthernet2/0 from LOADING to FULL, Loading Done
R4(config-router)#do write
Building configuration...
[OK]
```

---

### 🖥️ Router R5 (Area 1 Router & DHCP Server)

```text
R5#config t
Enter configuration commands, one per line.  End with CNTL/Z.
R5(config)#int g1/0
R5(config-if)#ip add 192.168.5.2 255.255.255.0
R5(config-if)#no shut
*Aug 21 12:29:00.931: %LINK-3-UPDOWN: Interface GigabitEthernet1/0, changed state to up
*Aug 21 12:29:01.931: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet1/0, changed state to up
R5(config-if)#int g2/0
R5(config-if)#ip add 192.168.7.1 255.255.255.0
R5(config-if)#no shut
*Aug 21 12:29:20.951: %LINK-3-UPDOWN: Interface GigabitEthernet2/0, changed state to up
*Aug 21 12:29:21.951: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet2/0, changed state to up
R5(config-if)#int lo0
R5(config-if)#ip add 5.5.5.5 255.255.255.255
*Aug 21 12:29:22.107: %LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback0, changed state to up

# Configure DHCP Server for Clients
R5(config)#ip dhcp pool ospf-pool
R5(dhcp-config)#network 192.168.7.0 255.255.255.0
R5(dhcp-config)#default-route 192.168.7.1
R5(dhcp-config)#dns 10.10.10.10
R5(dhcp-config)#exit

# Configure OSPF Process
R5(config)#router ospf 110
R5(config-router)#network 5.5.5.5 0.0.0.0 area 1
R5(config-router)#network 192.168.5.0 0.0.0.255 area 1
R5(config-router)#network 192.168.7.0 0.0.0.255 area 1
*Aug 21 12:48:50.015: %OSPF-5-ADJCHG: Process 110, Nbr 4.4.4.4 on GigabitEthernet1/0 from LOADING to FULL, Loading Done

# Test End-to-End Connectivity to ISP (8.8.8.8)
R5(config-router)#do ping 8.8.8.8
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 120/120/124 ms
R5(config-router)#do write
Building configuration...
[OK]
```

---

### 🖥️ Router R6 (Area 2 Internal Router)

```text
R6#config t
Enter configuration commands, one per line.  End with CNTL/Z.
R6(config)#int g1/0
R6(config-if)#ip add 192.168.4.2 255.255.255.0
R6(config-if)#no shut
*Aug 21 12:32:50.171: %LINK-3-UPDOWN: Interface GigabitEthernet1/0, changed state to up
*Aug 21 12:32:51.171: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet1/0, changed state to up
R6(config-if)#int g2/0
R6(config-if)#ip add 192.168.6.1 255.255.255.0
R6(config-if)#no shut
*Aug 21 12:33:05.775: %LINK-3-UPDOWN: Interface GigabitEthernet2/0, changed state to up
*Aug 21 12:33:06.775: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet2/0, changed state to up
R6(config-if)#int lo0
R6(config-if)#ip add 6.6.6.6 255.255.255.255
*Aug 21 12:33:09.151: %LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback0, changed state to up
R6(config-if)#exit

R6(config)#router ospf 110
R6(config-router)#network 6.6.6.6 0.0.0.0 area 2
R6(config-router)#network 192.168.4.0 0.0.0.255 area 2
R6(config-router)#network 192.168.6.0 0.0.0.255 area 2
*Aug 21 12:49:58.323: %OSPF-5-ADJCHG: Process 110, Nbr 3.3.3.3 on GigabitEthernet1/0 from LOADING to FULL, Loading Done
*Aug 21 12:51:12.927: %OSPF-5-ADJCHG: Process 110, Nbr 7.7.7.7 on GigabitEthernet2/0 from LOADING to FULL, Loading Done
R6(config-router)#do write
Building configuration...
[OK]
```

---

### 🖥️ Router R7 (Area 2 Internal Router)

```text
R7#config t
Enter configuration commands, one per line.  End with CNTL/Z.
R7(config)#int g1/0
R7(config-if)#ip add 192.168.6.2 255.255.255.0
R7(config-if)#no shut
*Aug 21 12:34:18.339: %LINK-3-UPDOWN: Interface GigabitEthernet1/0, changed state to up
*Aug 21 12:34:19.339: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet1/0, changed state to up
R7(config-if)#int lo0
R7(config-if)#ip add 7.7.7.7 255.255.255.255
*Aug 21 12:34:20.775: %LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback0, changed state to up
R7(config-if)#exit

R7(config)#router ospf 110
R7(config-router)#network 7.7.7.7 0.0.0.0 area 2
R7(config-router)#network 192.168.6.0 0.0.0.255 area 2
*Aug 21 12:51:13.451: %OSPF-5-ADJCHG: Process 110, Nbr 6.6.6.6 on GigabitEthernet1/0 from LOADING to FULL, Loading Done

# Verify Ping to External Destination (8.8.8.8)
R7(config-router)#do ping 8.8.8.8
Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 120/123/124 ms
R7(config-router)#do write
Building configuration...
[OK]
```

---

### 🖥️ Router R8 (External BGP ISP - AS 200)

```text
R8#config t
Enter configuration commands, one per line.  End with CNTL/Z.
R8(config)#int g1/0
R8(config-if)#ip add 192.168.20.2 255.255.255.0
R8(config-if)#no shut
*Aug 21 12:37:18.495: %LINK-3-UPDOWN: Interface GigabitEthernet1/0, changed state to up
*Aug 21 12:37:19.495: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet1/0, changed state to up
R8(config-if)#int lo0
R8(config-if)#ip add 8.8.8.8 255.255.255.255
*Aug 21 12:37:20.051: %LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback0, changed state to up
R8(config-if)#exit

R8(config)#router bgp 200
R8(config-router)#neighbor 192.168.20.1 remote-as 100
R8(config-router)#network 8.8.8.8 mask 255.255.255.255
*Aug 21 13:00:36.359: %BGP-5-ADJCHANGE: neighbor 192.168.20.1 Up
R8(config-router)#do write
Building configuration...
[OK]
```

---

### 💻 VPCS Client (PC1)

```text
PC1> sh ip
NAME        : PC1[1]
IP/MASK     : 192.168.7.2/24
GATEWAY     : 192.168.7.1
DNS         : 10.10.10.10
DHCP SERVER : 192.168.7.1
DHCP LEASE  : 84415, 86400/43200/75600
MAC         : 00:50:79:66:68:00

PC1> ping 5.5.5.5
84 bytes from 5.5.5.5 icmp_seq=1 ttl=255 time=15.473 ms
84 bytes from 5.5.5.5 icmp_seq=2 ttl=255 time=16.520 ms

PC1> ping 8.8.8.8
84 bytes from 8.8.8.8 icmp_seq=1 ttl=251 time=138.404 ms
84 bytes from 8.8.8.8 icmp_seq=2 ttl=251 time=137.826 ms
84 bytes from 8.8.8.8 icmp_seq=3 ttl=251 time=137.447 ms
84 bytes from 8.8.8.8 icmp_seq=4 ttl=251 time=138.579 ms
84 bytes from 8.8.8.8 icmp_seq=5 ttl=251 time=136.134 ms

PC1> save
Saving startup configuration to startup.vpc
.  done
```

---

## 🧪 Verification Commands & Summary

```bash
# Check OSPF Neighbors and Adjacencies
R1# show ip ospf neighbor

# Check OSPF Routing Table and Inter-Area / External Routes (O E2)
R4# show ip route ospf

# Check BGP Summary & Neighbors on ASBR (R1) and ISP (R8)
R1# show ip bgp summary
R8# show ip bgp

# Test End-to-End Reachability to Remote ISP Destination
R5# ping 8.8.8.8
R7# ping 8.8.8.8
PC1> ping 8.8.8.8
```

---

## 📁 Repository Path
- Topology file: `Seventh_Lab_OSPF&BGP/Multi-Area-OSPF-eBGP-Redistribution-Lab.gns3`
- Config files: `Seventh_Lab_OSPF&BGP/project-files/dynamips/`
- Topology Image: `Seventh_Lab_OSPF&BGP/images/topology.png`

---

## 📝 Author

**Chakresh Ram Kudupudi**  
Cybersecurity & Computer Networking Enthusiast  
GNS3 | Cisco IOS | OSPF Multi-Area Routing
