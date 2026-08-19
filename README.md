# 🌐 Networking Labs (GNS3)

Welcome to the **Networking Labs** repository! This project contains hands-on network simulation labs created with [GNS3](https://www.gns3.com/), covering core computer networking concepts, Cisco IOS configuration, packet analysis, and network security protocols.

---

## 📁 Repository Structure

```text
Networking-Labs/
├── First_Ping_Lab/          # Lab 1: Basic IP Configuration & ICMP Ping Verification
│   ├── First_Lab.gns3       # GNS3 Topology file
│   └── project-files/       # Device startup configs, VPCS state, packet captures (.pcap)
│
├── Second_SSH_Lab/          # Lab 2: SSH Configuration & Secure Remote Management
│   ├── Second_Lab.gns3      # GNS3 Topology file
│   └── project-files/       # Router startup configs & SSH key parameters
│
├── Third_DHCP_Lab/          # Lab 3: DHCP Configuration & DORA Packet Analysis
│   ├── Third_DHCP_Lab.gns3  # GNS3 Topology file
│   ├── README.md            # Lab documentation with screenshots
│   └── project-files/       # Router startup configs & packet captures (.pcap)
│
├── Forth_SBI_BANK_PROJECT/  # Lab 4: Static Routing + DHCP + Redundant Path Lab
│   ├── Forth_SBI_Bank.gns3  # GNS3 Topology file
│   ├── README.md            # Lab documentation with topology screenshot
│   ├── images/              # Topology screenshot (images/topology.png)
│   └── project-files/       # Router startup configs & VPCS state
│
├── Fifth_OSPF/              # Lab 5: Single-Area OSPF Dynamic Routing Configuration & Verification
│   ├── Test_OSPF.gns3       # GNS3 Topology file
│   ├── README.md            # Lab documentation with CLI screenshots & topology
│   ├── images/              # Topology & CLI screenshots
│   └── project-files/       # Router startup configs
│
├── Sixth_Lab_OSPF/          # Lab 6: Multi-Area OSPF Dynamic Routing Configuration & ABR Verification
│   ├── Sixth_Lab_OSPF.gns3  # GNS3 Topology file
│   ├── README.md            # Lab documentation with CLI & database screenshots
│   ├── images/              # Topology, CLI & OSPF Database screenshots
│   └── project-files/       # Router startup configs
│
└── README.md                # Project documentation
```

---

## 🚀 Lab Overview

### 1️⃣ First Ping Lab (`First_Ping_Lab/`)
* **Objective**: Establish baseline IP connectivity between Cisco routers and Virtual PC Simulator (VPCS) instances, and analyze ICMP traffic using Wireshark.
* **Key Concepts**:
  - Cisco IOS Interface IP Addressing (`GigabitEthernet`, `FastEthernet`)
  - Admin state management (`no shutdown`)
  - Subnetting (`192.168.12.0/24`)
  - Verification using `show ip interface brief` and `ping` commands
  - Packet capturing (`.pcap`) between router and switch nodes.

### 2️⃣ Second SSH Lab (`Second_SSH_Lab/`)
* **Objective**: Configure and verify Secure Shell (SSH v2) on Cisco IOS devices for secure CLI administration over an IP network (`192.168.1.0/24`).
* **Key Concepts**:
  - Hostname and IP Domain Name configuration (`ip domain-name`)
  - RSA Crypto Key Generation (`crypto key generate rsa`)
  - Local User Authentication (`username <user> secret <pass>`)
  - Line VTY Configuration (`transport input ssh`, `login local`)
  - SSH client connection testing and version verification.

### 3️⃣ Third DHCP Lab (`Third_DHCP_Lab/`)
* **Objective**: Configure a Cisco router as a DHCP server, dynamically assign IP parameters to VPCS clients, and analyze the complete DORA process with Wireshark screenshots.
* **Key Concepts**:
  - Cisco IOS DHCP Server Configuration (`ip dhcp pool`)
  - Network address allocation, default gateway, and DNS options
  - IP Address Exclusions (`ip dhcp excluded-address`)
  - DORA Exchange Analysis (Discover → Offer → Request → Acknowledgment)
  - DHCP Verification commands (`show ip dhcp pool`, `show ip dhcp binding`)
  - Screenshots & Wireshark display filters (`dhcp`)

### 4️⃣ Fourth SBI Bank Project (`Forth_SBI_BANK_PROJECT/`)
* **Objective**: Build a redundant enterprise branch network topology connecting an SBI Bank router to virtual ISP paths (Airtel & Jio) and a simulated Google DNS (`8.8.8.8`) destination, featuring floating static routes, DHCP server allocation, and failover testing.
* **Key Concepts**:
  - Multi-router topology (`Google`, `Airtel`, `Jio`, `SBI-BANK`)
  - Static Routing & Default Routes (`ip route`)
  - Floating Static Routes using Administrative Distance (AD 10 Primary vs AD 20 Backup)
  - Cisco IOS DHCP Server (`ip dhcp pool SBI-BANK`) for client network (`192.168.5.0/24`)
  - Loopback interface configuration (`Loopback0 8.8.8.8/32`)
  - Failover testing & path verification (`ping`, `traceroute`, interface `shutdown` / `no shutdown`)
  - Topology screenshot included (`images/topology.png`)

### 5️⃣ Fifth OSPF Lab (`Fifth_OSPF/`)
* **Objective**: Configure and verify Single-Area OSPF (Open Shortest Path First) dynamic routing across a linear 3-router Cisco topology (`R1` ↔ `R2` ↔ `R3`), enabling dynamic route discovery and full loopback-to-loopback reachability.
* **Key Concepts**:
  - Link-State Dynamic Routing Protocol & Shortest Path First (SPF) algorithm
  - Single-Area OSPF Architecture (`router ospf 110`, `area 0`)
  - Loopback Interface Configuration (`Loopback0`) for Router ID and end-to-end reachability testing
  - Wildcard Masking (`0.0.0.255` for `/24` subnets, `0.0.0.0` for `/32` host routes)
  - OSPF Adjacency States and Neighbor Relationships (`LOADING` to `FULL`)
  - Verification & testing via `show ip ospf neighbor`, `show ip route ospf`, and `ping` commands
  - Comprehensive screenshots included (`images/topology.png`, `images/r1_cli.png`, etc.)

### 6️⃣ Sixth OSPF Lab (`Sixth_Lab_OSPF/`)
* **Objective**: Configure and verify Multi-Area OSPF (Open Shortest Path First) dynamic routing across a 4-router topology (`R1` ↔ `R2` ↔ `R3` ↔ `R4`), establishing an Area Border Router (`R3` ABR) between Backbone Area 0 and Standard Area 1 for inter-area route discovery and end-to-end reachability.
* **Key Concepts**:
  - Multi-Area OSPF Hierarchy (`Area 0` Backbone & `Area 1` Standard Area)
  - Router Roles: Internal Routers (IR) vs Area Border Router (ABR `R3`)
  - LSA Types & Inter-Area Summarization (Type 1/2 Intra-Area & Type 3 Summary LSAs)
  - Loopback Interface Configuration (`Loopback0`) for Router ID stability and host connectivity simulation
  - Wildcard Masking (`0.0.0.255` for `/24` subnets, `0.0.0.0` for `/32` host IPs)
  - Verification commands (`show ip ospf database`, `show ip ospf neighbor`, `show ip route ospf`, `ping`)
  - Comprehensive screenshots included (`images/topology.png`, `images/ospf_database.png`, CLI outputs for R1-R4)

---

## 🛠️ Prerequisites & Tools

To run and interact with these labs, you will need:
* **[GNS3 Environment](https://www.gns3.com/)**: Version 2.2.x or higher
* **Router Images**: Cisco IOS (Dynamips supported e.g. c7200 / c3725 / c2600)
* **Virtual PC Simulator (VPCS)**: Included with GNS3
* **Wireshark**: Integrated with GNS3 for live packet captures

---

## 📖 How to Open a Lab in GNS3

1. Clone this repository:
   ```bash
   git clone https://github.com/chakreshram11/Networking-Labs.git
   ```
2. Open **GNS3**.
3. Select **File > Open Project** (`Ctrl + O`).
4. Navigate into the specific lab folder (e.g., `First_Ping_Lab/`, `Second_SSH_Lab/`, `Third_DHCP_Lab/`, `Forth_SBI_BANK_PROJECT/`, `Fifth_OSPF/`, or `Sixth_Lab_OSPF/`).
5. Select the `.gns3` topology file.
6. Start the nodes and open the consoles to explore or modify device configurations.

---

## 📝 Author & License

Created by **[chakreshram11](https://github.com/chakreshram11)**.  
Distributed under the MIT License. Contributions and improvements are welcome!
