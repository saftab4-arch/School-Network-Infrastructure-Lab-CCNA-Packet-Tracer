# 🏫 School Network Infrastructure Lab — CCNA Packet Tracer

## Project Overview

A single-school campus network built from scratch in Cisco Packet Tracer, demonstrating real-world K-12 network design principles. This lab simulates a school building with an MDF (Main Distribution Frame) housing a Layer 3 core switch connected to an IDF (Intermediate Distribution Frame) access switch serving classrooms with wired teacher PCs and wireless student access via Access Points.

**This is Part 1 of a larger project.** The full build (coming soon) will include a 3-school district with a district office, WAN links, OSPF routing, printer VLANs, and security ACLs.

---

## Architecture

```
                    ┌───────────────┐
                    │     MDF       │
                    │  3560-24PS    │
                    │  (Layer 3)    │
                    │               │
                    │  SVI VLAN 10: 10.1.10.1 (Staff Gateway)
                    │  SVI VLAN 20: 10.1.20.1 (Student Gateway)
                    │  ip routing enabled
                    └───────┬───────┘
                            │
                      Trunk (Gi0/1)
                      802.1Q dot1q
                      Carries VLAN 10, 20
                            │
                    ┌───────┴───────┐
                    │     IDF-1     │
                    │  2960-24TT    │
                    │  (Layer 2)    │
                    │               │
                    │  Fa0/1 → Teacher-101 [VLAN 10]
                    │  Fa0/2 → AP-101      [VLAN 20]
                    │  Fa0/3 → Teacher-102 [VLAN 10]
                    │  Fa0/4 → AP-102      [VLAN 20]
                    └───────────────┘
                       /         \
                    ))))         ))))
                   /                 \
            Student-101          Student-102
            (Wireless)           (Wireless)
```

---

## Network Design Details

### Physical Infrastructure (Real-World Mapping)

| Component | Lab Device | Real-World Equivalent |
|-----------|-----------|----------------------|
| MDF Core Switch | Cisco 3560-24PS (L3) | Cisco Catalyst 9300 with SFP modules |
| IDF Access Switch | Cisco 2960-24TT (L2) | Cisco Catalyst 9200 in IDF closet |
| MDF ↔ IDF Link | Copper GigE trunk | Fiber with SFP transceivers via cable tray |
| Teacher PCs | PC-PT (wired) | Desktop connected via wall jack → in-wall Cat6 → patch panel → switch |
| Access Points | AccessPoint-PT | Cisco Meraki/Aruba AP mounted on classroom ceiling |
| Student Devices | Laptop-PT (wireless NIC: WPC300N) | Chromebooks connecting to Student-WiFi SSID |

### VLAN Design

| VLAN ID | Name | Purpose | Subnet | Gateway |
|---------|------|---------|--------|---------|
| 10 | STAFF | Teacher wired PCs | 10.1.10.0/24 | 10.1.10.1 |
| 20 | STUDENTS | Student wireless devices | 10.1.20.0/24 | 10.1.20.1 |

### IP Addressing Scheme

The addressing follows a scalable **10.SCHOOL.VLAN.HOST** format designed for multi-site expansion:

| Device | IP Address | VLAN | Connection |
|--------|-----------|------|------------|
| MDF-CORE SVI VLAN 10 | 10.1.10.1 | 10 | Gateway for Staff |
| MDF-CORE SVI VLAN 20 | 10.1.20.1 | 20 | Gateway for Students |
| Teacher-101 | 10.1.10.10 | 10 | Wired (Fa0/1) |
| Teacher-102 | 10.1.10.11 | 10 | Wired (Fa0/3) |
| Student-101 | 10.1.20.10 | 20 | Wireless (AP-101) |
| Student-102 | 10.1.20.11 | 20 | Wireless (AP-102) |

### Port Assignment (IDF-1 Switch)

| Port | Mode | VLAN | Description | Device |
|------|------|------|-------------|--------|
| Gi0/1 | Trunk (802.1Q) | All | Uplink to MDF-CORE | Core Switch |
| Fa0/1 | Access | 10 | Rm101-Teacher-PC | Teacher Desktop |
| Fa0/2 | Access | 20 | Rm101-WiFi-AP | Access Point |
| Fa0/3 | Access | 10 | Rm102-Teacher-PC | Teacher Desktop |
| Fa0/4 | Access | 20 | Rm102-WiFi-AP | Access Point |

---

## Key Concepts Demonstrated

### 1. MDF/IDF Hierarchy
- **MDF (Main Distribution Frame):** Central network room housing the Layer 3 core switch. In production, this is where the WAN fiber from the district terminates and all IDF fiber runs originate.
- **IDF (Intermediate Distribution Frame):** Remote switch closet closer to classrooms. Connected to MDF via trunk uplink. Houses Layer 2 access switches and patch panels.
- **Star Topology:** Every IDF connects directly back to the MDF — no daisy-chaining.

### 2. VLAN Segmentation
- Staff and student traffic are logically separated on the same physical switch.
- Each port is individually assigned to a VLAN via CLI configuration.
- Devices on different VLANs cannot communicate without Layer 3 routing.

### 3. Trunk Links (802.1Q)
- The IDF-to-MDF uplink is configured as an 802.1Q trunk.
- Trunk carries all VLAN traffic tagged with VLAN IDs.
- Access ports carry single untagged VLAN traffic.

### 4. Inter-VLAN Routing (Layer 3 SVIs)
- The MDF-CORE 3560 has `ip routing` enabled, making it a Layer 3 switch.
- SVIs (Switch Virtual Interfaces) created for each VLAN serve as default gateways.
- All inter-VLAN routing happens at the MDF core — IDF switches are Layer 2 only.

### 5. Wireless Integration
- Access Points connected to switch access ports on VLAN 20.
- Student laptops equipped with WPC300N wireless NICs.
- WiFi traffic enters the wired network on the student VLAN, fully separated from staff.

---

## Verification & Testing

### Test 1: Same-VLAN Communication (Staff → Staff)
- **Source:** Teacher-101 (10.1.10.10, VLAN 10)
- **Destination:** Teacher-102 (10.1.10.11, VLAN 10)
- **Result:** ✅ 4/4 replies, 0% loss, <1ms latency (wired)
- **TTL:** 128 (no router hop — switched within VLAN)

### Test 2: Same-VLAN Communication (Student → Student, Wireless)
- **Source:** Student-101 (10.1.20.10, VLAN 20)
- **Destination:** Student-102 (10.1.20.11, VLAN 20)
- **Result:** ✅ 4/4 replies, 0% loss, 13-33ms latency (wireless)
- **TTL:** 128 (no router hop — switched within VLAN)

### Test 3: Cross-VLAN Communication (Staff → Student)
- **Source:** Teacher-101 (10.1.10.10, VLAN 10)
- **Destination:** Student-101 (10.1.20.10, VLAN 20)
- **Result:** ✅ 4/4 replies, 0% loss, 7-13ms latency
- **TTL:** 127 (decremented by 1 — confirms traffic routed through MDF-CORE L3 switch)

### Traffic Flow — Cross-VLAN Ping Path:
```
Teacher-101 (VLAN 10) → IDF-1 Fa0/1 (access VLAN 10)
  → Trunk Gi0/1 (tagged VLAN 10)
  → MDF-CORE receives on trunk
  → SVI VLAN 10 (10.1.10.1) processes packet
  → ip routing: destination 10.1.20.10 is on VLAN 20
  → Routes to SVI VLAN 20 (10.1.20.1) [TTL decrements 128→127]
  → Trunk Gi0/1 back to IDF-1 (tagged VLAN 20)
  → IDF-1 Fa0/2 (access VLAN 20)
  → AP-101 → wireless → Student-101
  → Reply traverses the same path back
```

---

## Switch Configurations

<details>
<summary>IDF-1 (2960-24TT) — Click to expand</summary>

```
hostname IDF-1
no ip domain-lookup

vlan 10
 name STAFF

vlan 20
 name STUDENTS

interface GigabitEthernet0/1
 switchport mode trunk
 description Trunk-to-MDF-CORE

interface FastEthernet0/1
 switchport mode access
 switchport access vlan 10
 description Rm101-Teacher-PC

interface FastEthernet0/2
 switchport mode access
 switchport access vlan 20
 description Rm101-WiFi-AP

interface FastEthernet0/3
 switchport mode access
 switchport access vlan 10
 description Rm102-Teacher-PC

interface FastEthernet0/4
 switchport mode access
 switchport access vlan 20
 description Rm102-WiFi-AP
```
</details>

<details>
<summary>MDF-CORE (3560-24PS) — Click to expand</summary>

```
hostname MDF-CORE
no ip domain-lookup

ip routing

vlan 10
 name STAFF

vlan 20
 name STUDENTS

interface GigabitEthernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 description Trunk-to-IDF-1

interface vlan 10
 ip address 10.1.10.1 255.255.255.0
 description STAFF-Gateway
 no shutdown

interface vlan 20
 ip address 10.1.20.1 255.255.255.0
 description STUDENTS-Gateway
 no shutdown
```
</details>

---

## Real-World Production Notes

This lab uses copper connections and simplified devices for Packet Tracer compatibility. In a production school environment:

- **MDF ↔ IDF links** would use **fiber optic cables with SFP transceiver modules** running through cable trays above ceiling tiles and cable risers between floors.
- **Fiber terminates at patch panels** on both ends (MDF and IDF) — short fiber patch cords connect from the patch panel to SFP ports on the switch.
- **In-wall Cat6 cabling** runs from each classroom wall jack to a **copper patch panel** in the nearest IDF closet. Short patch cords connect from the patch panel to switch access ports.
- **Every cable, port, and jack is labeled** with room numbers and jack identifiers for documentation and troubleshooting.
- **Access Points** would be enterprise-grade (Cisco Meraki, Aruba) broadcasting **multiple SSIDs** on a trunk port — Staff-WiFi (VLAN 10), Student-WiFi (VLAN 20), Guest-WiFi (VLAN 50).
- **ACLs** would block student VLAN from reaching staff VLAN — students get internet-only access.
- **Port security and 802.1X** authentication would prevent unauthorized devices from connecting.

---

## Coming Next — Full District Build

The next phase expands this into a complete multi-site school district:

- 🏫 3 Schools (Primary, Elementary, High School) — each with MDF + 2 IDFs
- 🏢 District Office with core router and simulated internet
- 🌐 WAN links (fiber) connecting all schools to district
- 🔄 OSPF dynamic routing between all sites
- 🖨️ Printer VLAN (VLAN 60) with ACLs controlling print access
- 🔒 Security ACLs blocking student-to-staff cross-VLAN access
- 📋 Full IP addressing scheme: 10.SCHOOL.VLAN.HOST

---

## Tools Used

- Cisco Packet Tracer 8.x
- Cisco IOS CLI

## Author

**Syed Aftab**
