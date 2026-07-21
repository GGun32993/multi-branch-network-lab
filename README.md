# Multi-Branch Enterprise Network Topology & Security Lab (CCNA Level)

🌐 **Language / สลับภาษา:** [ 🇬🇧 English ](README.md) | [ 🇹🇭 ภาษาไทย (Thai) ](README.th.md)

---

![Topology Diagram](topology/topology-diagram.png)

## 📌 Project Overview
This repository contains the full architecture, device configurations, and verification artifacts for a **Multi-Branch Enterprise Network Lab** designed and verified in Cisco Packet Tracer. 

The project demonstrates enterprise-grade network engineering principles aligned with CCNA standards, covering VLAN segmentation, Inter-VLAN routing, dynamic single-area OSPF routing with load balancing, edge NAT/PAT translation, centralized cross-subnet DHCP relay, internal Intranet Web/DNS services, and zero-trust Extended Access Control Lists (ACLs).

---

## 🏗️ Network Architecture & IP Subnetting Scheme

### 1. Enterprise Subnetting Plan (`192.168.0.0/16` & `10.0.0.0/30`)
| Site / Link | Department / Function | VLAN ID | IP Network / Prefix | Gateway IP | Usable Range |
| :--- | :--- | :---: | :---: | :---: | :---: |
| **Head Office (HQ)** | Server Farm / Data Center | VLAN 1 (SVI) | `192.168.10.0/24` | `192.168.10.1` | `192.168.10.2` - `192.168.10.254` |
| **Branch A (Branch 1)**| Sales Department | VLAN 10 | `192.168.20.0/24` | `192.168.20.1` | `192.168.20.2` - `192.168.20.254` |
| | HR Department | VLAN 20 | `192.168.21.0/24` | `192.168.21.1` | `192.168.21.2` - `192.168.21.254` |
| | IT Department | VLAN 30 | `192.168.22.0/24` | `192.168.22.1` | `192.168.22.2` - `192.168.22.254` |
| **Branch B (Branch 2)**| Sales Department | VLAN 10 | `192.168.30.0/24` | `192.168.30.1` | `192.168.30.2` - `192.168.30.254` |
| | HR Department | VLAN 20 | `192.168.31.0/24` | `192.168.31.1` | `192.168.31.2` - `192.168.31.254` |
| | IT Department | VLAN 30 | `192.168.32.0/24` | `192.168.32.1` | `192.168.32.2` - `192.168.32.254` |
| **WAN Link 1** | HQ <----> Branch A | N/A | `10.0.0.0/30` | N/A | `10.0.0.1` (HQ) - `10.0.0.2` (Br A) |
| **WAN Link 2** | HQ <----> Branch B | N/A | `10.0.0.4/30` | N/A | `10.0.0.5` (HQ) - `10.0.0.6` (Br B) |
| **WAN Link 3** | Branch A <----> Branch B | N/A | `10.0.0.8/30` | N/A | `10.0.0.9` (Br A) - `10.0.0.10` (Br B)|
| **ISP Link** | HQ Edge <----> ISP | N/A | `200.1.1.0/30` | N/A | `200.1.1.2` (HQ) - `200.1.1.1` (ISP) |
| **ISP Internet** | Loopback Simulation | N/A | `8.8.8.8/32` | N/A | `8.8.8.8` |

---

## ⚡ Key Implemented Technologies & Features

### 1. VLAN Segmentation & Switch Port Security
- **VLAN Segmentation**: Isolated broadcast domains for VLAN 10 (Sales), VLAN 20 (HR), and VLAN 30 (IT).
- **Access Ports**: End-user host ports explicitly configured in Access mode to prevent VLAN Hopping attacks.
- **Trunk Ports**: Switch-to-Router interconnects configured with 802.1Q encapsulation (`switchport mode trunk`).

### 2. Inter-VLAN Routing (Router-on-a-Stick)
- Configured 802.1Q subinterfaces on `Branch1-R1` and `Branch2-R1` (`Gi0/0.10`, `Gi0/0.20`, `Gi0/0.30`).
- Encapsulated default gateways per VLAN (`encapsulation dot1Q <vlan>`).

### 3. Dynamic Routing (OSPF Area 0)
- Single-Area OSPF Area 0 deployed across all enterprise routers (`HQ-R1`, `Branch1-R1`, `Branch2-R1`).
- **Equal-Cost Multi-Path (ECMP)** load balancing automatically active over redundant `/30` WAN paths.
- Default route `0.0.0.0/0` injected dynamically to all branches via `default-information originate`.

### 4. Edge NAT/PAT & Internet Gateway
- **NAT Overload (PAT)** configured on `HQ-R1` (`ip nat inside source list 1 interface Gi0/0 overload`).
- Translates internal private IPv4 addresses (`192.168.0.0/16`) to a single public IP to reach Internet targets (`8.8.8.8`).

### 5. Enterprise Data Center & Centralized Services
- **Data Center Gateway**: Configured SVI `interface Vlan1` (`192.168.10.1`) via HWIC expansion switchport module on `HQ-R1`.
- **Intranet Web Portal**: Hosted HTTP/HTTPS Web Server on `HQ-Server` (`192.168.10.100`).
- **Internal DNS Server**: Configured DNS A-Record resolving `www.company.local` to `192.168.10.100`.
- **Centralized DHCP & Relay Agent**: Central DHCP pools on `HQ-Server` serving IP addresses automatically across subnets via `ip helper-address 192.168.10.100` configured on branch subinterfaces.

### 6. Zero-Trust Security Policy (Extended ACL)
- Inbound Extended Access List (`access-list 100`) applied on `Branch1-R1` (`Gi0/0.10` inbound).
- **Security Policy**: Denies Sales subnet (`192.168.20.0/24`) from reaching HR subnet (`192.168.21.0/24`), while permitting Intranet Web (`www.company.local`), DNS, and Internet access.

---

## 💡 Challenges & What I Learned
1. **OSPF Default Route Propagation**: Learned that edge routers (`HQ-R1`) must explicitly inject default routes into OSPF using `default-information originate` so branch routers can reach external targets.
2. **NAT Overload Interface Scoping**: Discovered that inbound WAN interfaces receiving inter-branch traffic (`Gi0/1`, `Gi0/2`) must be configured with `ip nat inside` for NAT translation to trigger correctly.
3. **ACL Wildcard Mask Precision**: Experienced how an overly broad `/16` wildcard mask (`0.0.255.255`) accidentally blocked server traffic. Correcting the mask to `/24` (`0.0.0.255`) successfully isolated the intended target.
4. **Power Cycle & Config Persistence**: Reinforced the essential habit of saving active configurations to NVRAM (`write memory`) prior to rebooting or adding HWIC expansion modules.

---

## 🧪 Verification & Test Results

| Test Case | Description | Source | Destination | Expected Result | Status | Screenshot Artifact |
| :--- | :--- | :--- | :--- | :--- | :---: | :--- |
| **1. Internet Access** | NAT/PAT & Default Route Ping | `PC-Sales-A` | `8.8.8.8` (ISP) | `Reply from 8.8.8.8` (0% Loss) | ✅ PASSED | `test-results/ping-internet-test.png` |
| **2. Server Farm Ping** | Inter-Branch OSPF Server Access | `PC-Sales-A` | `192.168.10.100` | `Reply from 192.168.10.100` | ✅ PASSED | `test-results/ping-server-test.png` |
| **3. Intranet Web Portal** | DNS A-Record & HTTP Web Access | `PC-Sales-A` | `www.company.local` | Web Portal Page Loaded | ✅ PASSED | `test-results/web-portal-test.png` |
| **4. Extended ACL Security** | Zero-Trust Inter-VLAN Block Policy | `PC-Sales-A` | `192.168.21.10` (HR) | `Destination host unreachable` | ✅ PASSED | `test-results/acl-block-test.png` |

### Test Result Visual Artifacts

#### Test 1: Internet Access via NAT/PAT (`ping 8.8.8.8`)
![Ping Internet](test-results/ping-internet-test.png)

#### Test 2: Data Center Server Access (`ping 192.168.10.100`)
![Ping Server](test-results/ping-server-test.png)

#### Test 3: Intranet Web Portal Access (`www.company.local`)
![Web Portal](test-results/web-portal-test.png)

#### Test 4: Extended ACL Security Block (`ping 192.168.21.10`)
![ACL Block Test](test-results/acl-block-test.png)

---

## 📂 Repository Structure

```text
multi-branch-network-lab/
├── README.md                                  # Primary project documentation (English Version)
├── README.th.md                               # Project documentation (Thai Version)
├── .gitignore                                 # Git ignore rules (.DS_Store, Thumbs.db, *.tmp, *.log)
├── topology/
│   └── topology-diagram.png                   # Full 3-Branch + Data Center + ISP network topology diagram
├── packet-tracer/
│   └── multi-branch-network-lab.pkt           # Completed Cisco Packet Tracer lab file
├── test-results/
│   ├── ping-internet-test.png                 # Verification screenshot: Internet Ping (8.8.8.8)
│   ├── ping-server-test.png                   # Verification screenshot: Server Ping (192.168.10.100)
│   ├── web-portal-test.png                    # Verification screenshot: Intranet Web Browser Portal
│   └── acl-block-test.png                     # Verification screenshot: Extended ACL Security Block
├── configs/
│   ├── HQ-R1.txt                              # Head Office Edge Router Running Config
│   ├── Branch1-R1.txt                         # Branch A Router Running Config (RoAS/OSPF/DHCP Relay/ACL 100)
│   ├── Branch1-SW1.txt                        # Branch A Access Switch Running Config (VLANs/Trunk)
│   ├── Branch2-R1.txt                         # Branch B Router Running Config (RoAS/OSPF/DHCP Relay)
│   ├── Branch2-SW1.txt                        # Branch B Access Switch Running Config (VLANs/Trunk)
│   └── ISP-R1.txt                             # ISP Router Running Config (Loopback 8.8.8.8)
└── docs/                                      # Additional documentation folder
```

---

## 🚀 How to Run the Lab in Cisco Packet Tracer

1. Clone this repository:
   ```bash
   git clone https://github.com/GGun32993/multi-branch-network-lab.git
   ```
2. Open **Cisco Packet Tracer** (version 8.0 or newer).
3. Open the project file: `packet-tracer/multi-branch-network-lab.pkt`.
4. Allow 30-40 seconds for OSPF adjacencies and STP ports to converge.
5. Test from any branch PC (e.g., `PC-Sales-A`):
   - Switch IP configuration to **DHCP** to receive an IP automatically.
   - Open **Command Prompt** and test: `ping 8.8.8.8` or `ping 192.168.10.100`.
   - Open **Web Browser** and test: `www.company.local`.
