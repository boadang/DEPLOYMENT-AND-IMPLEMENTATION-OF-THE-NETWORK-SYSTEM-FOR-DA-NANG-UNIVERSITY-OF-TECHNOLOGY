# Enterprise Campus Network Design and Implementation for Da Nang University of Technology

## 1. Executive Summary & Project Introduction
* **Project Name:** Deployment and Implementation of the Network System for Da Nang University of Technology
* **Project Contributors:** Dang Quoc Bao & Team 1
* **Course:** Switching and Routing (1)
* **Supervising Instructor:** M.Sc. Le Tu Thanh

### Background & Rationale
In the era of educational digital transformation, a robust, secure, and highly available network infrastructure forms the backbone of academic excellence, scientific research, and administrative efficiency. Da Nang University of Technology (DUT) operates on a massive institutional scale, accommodating nearly 18,000 students and postgraduates, over 546 faculty members and staff, 14 specialized faculties, 9 functional departments, and 11 advanced research centers. 

Managing such an expansive user base requires an Enterprise-grade Campus Network Infrastructure. The traditional monolithic network models fail to meet these demands, resulting in broadcast storms, severe bottlenecks, security vulnerabilities, and single points of failure (SPOF). This project delivers a comprehensive blueprint and virtual implementation of a high-performance, resilient (24/7/365 available), and secure Three-Tier Campus Network tailored specifically to the structural and operational demands of DUT.

---

## 2. Project Objectives

### General Objective
To research, analyze, design, and virtually deploy an end-to-end Enterprise Campus Network architecture using industry-standard switching and routing paradigms to maximize throughput, guarantee high availability, and enforce multi-layered cybersecurity boundaries.

### Specific Technical Objectives
* **Hierarchical Topology Engineering:** Design a structural hierarchical topology to effectively isolate traffic, eliminate network congestion, and optimize inter-building bandwidth allocation.
* **Logical Domain Segmentation:** Implement Virtual Local Area Networks (VLANs) and 802.1Q trunking encapsulation to restrict broadcast domains and optimize bandwidth utilization.
* **Dynamic Route Optimization:** Deploy and fine-tune a dynamic routing protocol capable of sub-second convergence, automatic metric calculations, and efficient scaling.
* **High Availability & First-Hop Redundancy:** Eliminate Single Points of Failure at the distribution and gateway layers using physical link aggregation and virtual gateway redundancy protocols.
* **Multi-Layered Security Enforcement:** Embed strict security primitives across Layer 2 (Access) to Layer 4 (Transport/App) using Access Control Lists (ACLs), hardware Firewalls, and endpoint port-level mitigations.
* **Simulative Verification:** Validate all protocol operations, failover behaviors, and traffic policy enforcements inside a virtualized network environment.

---

## 3. Network Design & Architecture

The network employs the **Cisco Three-Tier Hierarchical Model** (Core, Distribution, Access), which decouples complex network tasks into specialized layers, enabling seamless scalability and structured troubleshooting.

              +-----------------------+
              |  Internet / ISP WAN   |
              +-----------+-----------+
                          |
                    [ Firewall ]
                          |
              +-----------+-----------+
              |   Edge Router (NAT)   |
              +-----------+-----------+
                          |
        +-----------------+-----------------+
        |                                   |
+---------+---------+               +---------+---------+
|  Core L3 Switch A |==============>|  Core L3 Switch B |  <-- CORE LAYER (OSPF Area 0)
+---------+---------+  EtherChannel +---------+---------+
|        \                 /        |
|         \               /         |
+---------+---------+  \         /  +---------+---------+
| Dist. L3 Switch A |   X       X   | Dist. L3 Switch B |  <-- DISTRIBUTION LAYER (HSRP / Inter-VLAN)
+---------+---------+  /         \  +---------+---------+
|         /               \         |
|        /                 \        |
+---------+---------+               +---------+---------+
| Access Switch 01  |               | Access Switch 02  |  <-- ACCESS LAYER (VLANs, Port Security)
+---------+---------+               +---------+---------+
|       |       |                   |       |       |
[PC]   [VoIP]   [AP]                [PC]    [Srv]   [Cam]  <-- ENDPOINTS


### 3.1. Layered Functional Specifications
* **Core Layer (The Backbone):** Built utilizing ultra-high-throughput Layer 3 Switches (e.g., Cisco Catalyst 9500 series specification). This layer is purely dedicated to high-speed, low-latency packet switching across the campus core. No packet filtering, ACLs, or terminal host distribution happen here to prevent CPU overhead. It runs Multi-Area OSPF (Area 0) for core routing and Cross-Stack EtherChannel links for multi-gigabit core trunking.
* **Distribution Layer (The Policy Engine):** Acts as the boundary between Layer 2 switching and Layer 3 routing. These Layer 3 multi-layer switches handle Inter-VLAN routing via Switched Virtual Interfaces (SVIs). This layer processes Access Control Lists (ACLs), manages Quality of Service (QoS) markings for VoIP/Video traffic, aggregates Access-layer uplinks, and implements **Hot Standby Router Protocol (HSRP)** to serve as a redundant virtual default gateway.
* **Access Layer (The Edge Connection):** Deploys high-density Layer 2 Managed Switches (e.g., Cisco Catalyst 9200/2960-X series specification) to provide physical network ingress points for end devices (Workstations, Faculty Laptops, IP Phones, Surveillance Cameras, and Wireless Access Points). Key mechanisms enabled include 802.1Q trunking, Power over Ethernet (PoE), and endpoint-level defense mechanisms.

### 3.2. Logical Segmentation & Subnetting (VLAN Scheme)
To enforce the principle of least privilege and minimize network overhead, the campus is structurally divided into logical subnets:

| VLAN ID | Subnet Name | Description / Target Audience | Security Level |
| :--- | :--- | :--- | :--- |
| **VLAN 10** | `Management` | Network infrastructure management (SSH/HTTPS access to Switched/Routers) | Critical / Isolated |
| **VLAN 20** | `Administration` | Board of Directors, Finance, HR, Functional Departments | High |
| **VLAN 30** | `Faculty_Staff` | Professors, Lecturers, Research Labs, Academic Deans | Medium-High |
| **VLAN 40** | `Student_Labs` | Fixed Desktop Computer Rooms, General Student Workstations | Low (Monitored) |
| **VLAN 50** | `Wireless_Student`| Wi-Fi infrastructure allocated to students via WLC authentication | Low (Restricted) |
| **VLAN 60** | `VoIP_Phones` | IP Telephony infrastructure across campus (Prioritized via Voice QoS) | Medium |
| **VLAN 70** | `CCTV_Surveillance`| IP Cameras and Network Video Recorders (NVR) | Medium-Isolated |
| **VLAN 100**| `Server_DMZ` | Enterprise services: Student Portal, LMS, Website, Internal DNS/DHCP | High Protection |

---

## 4. Employed Technologies & Network Protocols

### Core Routing and Switching Engineering
1. **OSPF v2 (Open Shortest Path First):** Configured as the core Interior Gateway Protocol (IGP). Implemented as a Multi-Area design to reduce SPF algorithm calculations and optimize routing tables. Link-state advertisements (LSAs) are strictly tuned using passive-interfaces on user-facing links to optimize CPU cycles.
2. **HSRP (Hot Standby Router Protocol):** Configured across pairs of Distribution L3 switches. Switch A is designated as `Active` (Priority 110) and Switch B as `Standby` (Priority 100) with `preempt` enabled. This guarantees an automatic, sub-second failover of the default gateway if a distribution node collapses.
3. **Link Aggregation (LACP / EtherChannel):** Multiple physical ports are bundled into logical `Port-Channels` using the Link Aggregation Control Protocol (IEEE 802.3ad). This increases backbone inter-switch bandwidth (up to 40Gbps/100Gbps logical links) and prevents loop-induction topologies from disabling paths.
4. **IEEE 802.1Q & Per-VLAN Spanning Tree Plus (PVST+):** Ensures a loop-free topology while optimizing link utilization by establishing distinct Spanning Tree root bridges per individual VLAN.

### Security and Infrastructure Controls
5. **Port Security:** Configured on Access-layer ports to bind specific MAC addresses to a physical switchport. Ports are configured with a maximum MAC limit of 1 and a violation mode set to `shutdown` to mitigate MAC Flooding and Rogue Device Attachment.
6. **BPDU Guard & Root Guard:** Enabled on all Access Edge ports. If an unauthorized switch is attached and sends a Bridge Protocol Data Unit (BPDU), BPDU Guard instantly transitions the interface into an `err-disabled` state, protecting the Spanning Tree topology from hijacking.
7. **DHCP Snooping & Dynamic ARP Inspection (DAI):** Build an internal IP-to-MAC binding database by monitoring DHCP transactions. Untrusted ports are blocked from sourcing DHCP Offers (mitigating Rogue DHCP server attacks) and untrusted ARP requests are validated against the snooping database to eradicate Man-in-the-Middle (MitM) ARP Poisoning attacks.
8. **Extended Access Control Lists (ACLs):** Layer 3 and Layer 4 packet filters applied inbound and outbound at the Distribution layer to enforce inter-departmental segregation (e.g., denying Student VLANs from initiating SSH connections to the Management VLAN or accessing Server DMZ administrative ports).
9. **Stateful Firewall Integration & NAT/PAT:** Regulates traffic at the network perimeter. Port Address Translation (PAT) maps thousands of Private RFC 1918 internal IP addresses to a limited pool of Public IPv4 addresses on the Edge Router link.
10. **Cisco Wireless LAN Controller (WLC) Architecture:** Centralized CAPWAP tunneling architectures manage and provision all lightweight Access Points (LAPs) distributed across lecture halls, streamlining global SSID mapping and RF management.

---

## 5. Reference Configuration Snippets (Technical Showcase)

Here are samples of the configuration scripts implemented during the deployment phase:

### OSPF and EtherChannel on Core/Distribution Switches
! --- Configure EtherChannel (LACP) ---
interface Range GigabitEthernet0/1 - 2
 channel-group 1 mode active
 description Channel-Link-To-Core
!
interface Port-channel 1
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
! --- Configure OSPF Routing Protocol ---
router ospf 100
 router-id 1.1.1.1
 log-adjacency-changes
 network 10.1.0.0 0.0.255.255 area 0
 network 192.168.100.0 0.0.0.255 area 0
 passive-interface GigabitEthernet0/3
HSRP Gateway Redundancy Configuration (Distribution Layer)
Đoạn mã
! --- SVI Configuration on Dist-Switch-A (Active Primary Gateway) ---
interface Vlan 20
 ip address 10.20.0.2 255.255.255.0
 standby 20 ip 10.20.0.1
 standby 20 priority 110
 standby 20 preempt
 standby 20 authentication Md5DUTNet
Access Layer Port Hardening & Security
Đoạn mã
! --- Hardening End-User Access Ports --- !
interface Range FastEthernet0/1 - 24
 switchport mode access
 switchport access vlan 40
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation shutdown
 switchport port-security aging time 10
 spanning-tree portfast
 spanning-tree bpduguard enable
 ip dhcp snooping limit rate 20
6. Implementation, Verification & Test Results
Simulation Platform
The structural implementation, path checking, and configurations were built and extensively validated using Cisco Packet Tracer.

Empirical Verification Results
Dynamic DHCP Allocation Verification: Endpoint test machines across different buildings successfully initiated DHCP Discover packets. The Distribution layer successfully relayed packets to the central pool, resulting in accurate IP, Subnet Mask, Default Gateway, and DNS Server mapping across all VLANs.

End-to-End Connectivity (ICMP/Traceroute Checking): Complete network convergence was achieved. Endstations verified full reachability using ping and traceroute commands. Round-trip times (RTT) stabilized across internal paths. Packets destined for external public web services successfully traversed the Edge Router via NAT/PAT translation.

Failover Resilience Testing (High Availability): To simulate a catastrophic structural failure, the primary active Link Aggregation path and the master Distribution Switch were manually disabled (shutdown).

Result: HSRP successfully detected missing hellos from the active router, promoting the standby peer to Active status in less than 3 seconds. OSPF re-calculated path costs dynamically, rerouting core data streams with minimal packet loss.

Security Policy Enforcement Verification: Attempted unauthorized access configurations (e.g., Student node attempting to ping the Financial Database Server) were systematically dropped at the distribution interface. Packet analyzer utilities inside the simulator confirmed that Extended ACL rules caught and discarded the forbidden traffic packets (Communication administratively prohibited).

Layer 2 Attack Mitigation Testing: Attaching a secondary rogue switch attempting to inject superior configuration BPDUs resulted in the immediate shutdown of the interface by BPDU Guard, safely containing the infrastructure loop vector.

7. Future Roadmap & Strategic Upgrades
Core Bandwidth Upgrades: Planning transitions to next-generation hardware frameworks supporting 100 Gbps and 400 Gbps fiber backbones to support massive telemetry data transfers and concurrent high-definition distance learning streams.

Transition to Software-Defined Networking (SDN): Introduce Cisco DNA Center and an SD-Access overlay architecture, replacing legacy box-by-box CLI provisioning with policy-based automated workflows.

SD-WAN Border Integration: Implement Software-Defined Wide Area Networking (SD-WAN) edge routers to safely load-balance internet traffic across redundant ISP lines, prioritizing critical research traffic over standard web browsing.

Zero Trust Architecture (ZTA) & 802.1X: Integrate Cisco Identity Services Engine (ISE) to implement strict network identity validation. Every device connecting via Wi-Fi or Ethernet must authenticate dynamically, assigning granular, identity-based micro-segmentation tags instead of static VLAN IDs.
