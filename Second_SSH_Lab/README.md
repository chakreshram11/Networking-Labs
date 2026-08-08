# 🔐 Second SSH Lab — Cisco IOS SSH Configuration & Remote Management

This directory contains the GNS3 project files and complete step-by-step documentation for **Lab 2: Second SSH Lab**. This lab demonstrates how to configure Secure Shell (SSH v2) on Cisco IOS routers for encrypted, secure CLI remote administration over an IP network.

---

## 📋 Table of Contents
1. [Network Topology](#1-network-topology)
2. [SSH Prerequisites & Requirements](#2-ssh-prerequisites--requirements)
3. [Step-by-Step SSH Configuration Guide](#3-step-by-step-ssh-configuration-guide)
4. [Verification & Connecting via SSH](#4-verification--connecting-via-ssh)
5. [Complete Command Summary](#5-complete-command-summary)
6. [Configuration Flow Diagram](#6-configuration-flow-diagram)
7. [Technical Notes & Security Best Practices](#7-technical-notes--security-best-practices)

---

## 1. Network Topology

```text
              192.168.1.1/24                 192.168.1.2/24

              +------------+                 +------------+
              | R1         |                 | R2         |
              | (dhakresh) |-----------------|(dhittibabu)|
              +------------+                 +------------+
               g1/0                           g1/0
```

* **Router 1 (`R1` / Hostname: `dhakresh`)**: IP Address `192.168.1.1/24` on `GigabitEthernet1/0`
* **Router 2 (`R2` / Hostname: `dhittibabu`)**: IP Address `192.168.1.2/24` on `GigabitEthernet1/0`

---

## 2. SSH Prerequisites & Requirements

For SSH to operate successfully on a Cisco IOS device, the following conditions must be met:
1. **IP Connectivity**: Active IP address configured on an interface (`no shutdown`).
2. **Device Hostname**: A unique hostname (other than default `Router`).
3. **IP Domain Name**: Configured via `ip domain-name <domain>` (required for FQDN RSA key pair creation).
4. **RSA Key Pair**: Key modulus generated using `crypto key generate rsa` (minimum 768 bits for SSH v2; 1024+ bits recommended).
5. **Local User Database**: At least one local username and password configured for AAA/local authentication.
6. **VTY Line Configuration**: VTY lines configured with `transport input ssh` and `login local`.
7. **Privileged EXEC Password**: `enable secret` set so remote users can elevate to enable mode.

---

## 3. Step-by-Step SSH Configuration Guide

### Step 1: Configure Interface & IP Address (`R2`)
```text
R2# config t
Enter configuration commands, one per line. End with CNTL/Z.

R2(config)# interface g1/0
R2(config-if)# ip add 192.168.1.2 255.255.255.0
R2(config-if)# no shut
R2(config-if)# exit
```
*Console Messages:*
```text
*Aug  6 17:27:17.187: %LINK-3-UPDOWN: Interface GigabitEthernet1/0, changed state to up
*Aug  6 17:27:18.187: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet1/0, changed state to up
```

---

### Step 2: Set Hostname & IP Domain Name
```text
R2(config)# hostname dhittibabu
dhittibabu(config)# ip domain-name kakinada
```

---

### Step 3: Generate RSA Encryption Keys
```text
dhittibabu(config)# crypto key generate rsa
```
*Output:*
```text
The name for the keys will be: dhittibabu.kakinada

Choose the size of the key modulus in the range of 360 to 4096 for your
General Purpose Keys. Choosing a key modulus greater than 512 may take
a few minutes.

How many bits in the modulus [512]: 1024
% Generating 1024 bit RSA keys, keys will be non-exportable...

[OK] (elapsed time was 1 seconds)

*Aug  6 17:28:02.207: %SSH-5-ENABLED: SSH 1.99 has been enabled
```

---

### Step 4: Configure VTY Lines for SSH Access
```text
dhittibabu(config)# line vty 0 2
dhittibabu(config-line)# transport input ssh
dhittibabu(config-line)# transport output ssh
dhittibabu(config-line)# login local
dhittibabu(config-line)# exit
```

#### Command Breakdown:
* `line vty 0 2`: Selects Virtual Terminal lines 0 through 2.
* `transport input ssh`: Enforces SSH as the only incoming protocol (disables insecure Telnet).
* `transport output ssh`: Allows outgoing SSH sessions from this device.
* `login local`: Instructs the router to authenticate incoming sessions against the local user database.

---

### Step 5: Configure Local User & Enable Secret
```text
dhittibabu(config)# username admin password admin123
dhittibabu(config)# enable secret admin
```
* **Username**: `admin`
* **Password**: `admin123`
* **Enable Secret**: `admin`

---

### Step 6: Save Configuration to NVRAM
```text
dhittibabu(config)# do write
```
*Output:*
```text
Building configuration...
[OK]
```

---

## 4. Verification & Connecting via SSH

### Connecting from `R2` (`dhittibabu`) to `R1` (`dhakresh`)
Initiate an outbound SSH connection to R1's IP address (`192.168.1.1`):

```text
dhittibabu# ssh -l admin 192.168.1.1
Password:
```

### Elevated Privilege Access:
```text
dhakresh> enable
Password: 
dhakresh#
```

The prompt change from `dhakresh>` (User EXEC) to `dhakresh#` (Privileged EXEC) confirms successful authentication and authorization over an encrypted SSH connection!

---

## 5. Complete Command Summary

```text
! Step 1: Configure Interface
R2# config t
R2(config)# interface g1/0
R2(config-if)# ip add 192.168.1.2 255.255.255.0
R2(config-if)# no shut

! Step 2: Set Hostname & Domain Name
R2(config-if)# hostname dhittibabu
dhittibabu(config)# ip domain-name kakinada

! Step 3: Generate RSA Key Pair
dhittibabu(config)# crypto key generate rsa
! (Select 1024 or 2048 modulus bits)

! Step 4: Configure VTY Lines
dhittibabu(config)# line vty 0 2
dhittibabu(config-line)# transport input ssh
dhittibabu(config-line)# transport output ssh
dhittibabu(config-line)# login local
dhittibabu(config-line)# exit

! Step 5: User & Secret Credentials
dhittibabu(config)# username admin password admin123
dhittibabu(config)# enable secret admin

! Step 6: Save Configuration
dhittibabu(config)# do write

! Step 7: SSH Client Test (Connect to R1)
dhittibabu# ssh -l admin 192.168.1.1
```

---

## 6. Configuration Flow Diagram

```text
+------------------------+
|   1. IP Interface      |
| (ip add & no shutdown) |
+------------------------+
            │
            ▼
+------------------------+
| 2. Hostname & Domain   |
|  (hostname & domain)   |
+------------------------+
            │
            ▼
+------------------------+
| 3. RSA Key Generation  |
| (crypto key gen rsa)   |
+------------------------+
            │
            ▼
+------------------------+
|   4. SSH Enabled       |
|  (%SSH-5-ENABLED)      |
+------------------------+
            │
            ▼
+------------------------+
| 5. Line VTY Settings   |
| (transport input ssh & |
|      login local)      |
+------------------------+
            │
            ▼
+------------------------+
| 6. User & Enable Pass  |
|  (username & secret)   |
+------------------------+
            │
            ▼
+------------------------+
|  7. Save Configuration |
|       (do write)       |
+------------------------+
            │
            ▼
+------------------------+
|  8. Test Remote Access |
| (ssh -l admin <ip>)    |
+------------------------+
```

---

## 7. Technical Notes & Security Best Practices

> [!TIP]  
> **Key Security Guidelines**:
> - **Inbound vs Outbound Transport**:
>   - `transport input ssh`: Vital for restricting **incoming** VTY remote access to SSH only (blocking unencrypted Telnet).
>   - `transport output ssh`: Controls **outgoing** telnet/SSH commands executed from the router CLI.
> - **Password Hashing**: Use `username admin secret <pass>` or `service password-encryption` in production environments instead of plain-text `password` keyword.
> - **RSA Modulus Size**: Use **2048 bits** in modern production environments to satisfy SSH v2 security standards and compliance requirements.
