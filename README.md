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
4. Navigate into the specific lab folder (e.g., `First_Ping_Lab/` or `Second_SSH_Lab/`).
5. Select the `.gns3` topology file (e.g., `First_Lab.gns3` or `Second_Lab.gns3`).
6. Start the nodes and open the consoles to explore or modify device configurations.

---

## 📝 Author & License

Created by **[chakreshram11](https://github.com/chakreshram11)**.  
Distributed under the MIT License. Contributions and improvements are welcome!
