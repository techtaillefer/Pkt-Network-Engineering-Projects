# Enterprise Network Design and Implementation – Four-Story Branch Office

## Project Overview

In this more larger project, I designed and implemented a complete enterprise network for a four-story banking office from scratch. The network was modeled using a **hierarchical network design** and implemented in **Cisco Packet Tracer** with separate VLANs for every department, centralized DHCP, inter-VLAN routing, OSPF dynamic routing, wireless access, SSH management, port security, and dedicated enterprise servers.

The design uses **access, distribution, and core layers** to create a scalable network while also providing redundant paths between major networking devices. Each department operates on its own VLAN and IPv4 subnet, while multilayer switches provide inter-VLAN routing and OSPF distributes routes throughout the network.

---

## Project Objectives

The main objectives of this project were to:

- Design a hierarchical enterprise network.
- Separate departments using VLANs.
- Create an IPv4 subnetting plan from `192.168.10.0`.
- Support approximately 60 wired and wireless hosts per department.
- Configure inter-VLAN routing using multilayer switches.
- Configure centralized DHCP services.
- Use DHCP relay to provide addresses across multiple VLANs.
- Configure OSPF for dynamic route advertisement.
- Provide redundant connections between the access, distribution, and core layers.
- Configure secure SSH remote administration.
- Configure switch port security using sticky MAC addresses.
- Deploy departmental wireless access points.
- Configure HTTP and email services.
- Verify end-to-end connectivity between all departments.

---

# Network Topology

The topology follows a three-layer hierarchical model (Which is: Core, Distribution, Access):

<img width="1279" height="591" alt="bank-network-project-topology" src="https://github.com/user-attachments/assets/4696f91e-0f95-4cf3-9ed4-95484b589a32" />


The four routers form the CORE of the topology and provide redundant routed paths between the floors.

Multilayer switches operate at the DISTRIBUTION layer and provide:

- Layer 3 routing
- VLAN gateways
- Inter-VLAN routing
- DHCP relay
- OSPF connectivity to the routers

ACCESS switches connect:

- PCs
- Printers
- Wireless access points
- Servers

---

# Department Layout

## First Floor

| Department | PCs | Printers | VLAN |
|---|---:|---:|---:|
| Management | 20 | 4 | 10 |
| Research | 20 | 4 | 20 |
| Human Resources | 20 | 4 | 30 |

## Second Floor

| Department | PCs | Printers | VLAN |
|---|---:|---:|---:|
| Marketing | 20 | 4 | 40 |
| Accounting | 20 | 4 | 50 |
| Finance | 20 | 4 | 60 |

## Third Floor

| Department | PCs | Printers | VLAN |
|---|---:|---:|---:|
| Logistics / Store | 20 | 4 | 70 |
| Customer Care | 20 | 4 | 80 |
| Guest Area | 40 | 2 | 90 |

## Fourth Floor

| Department | PCs | Printers | VLAN |
|---|---:|---:|---:|
| Administration | 20 | 2 | 100 |
| ICT | 20 | 2 | 110 |
| Server Room | 2 Admin PCs | — | 120 |

The server room also contains:

- DHCP Server
- Email Server
- HTTP/HTTPS Server

---

# VLAN Plan

Each department is assigned a unique VLAN.

| VLAN | Department |
|---:|---|
| 10 | Management |
| 20 | Research |
| 30 | Human Resources |
| 40 | Marketing |
| 50 | Accounting |
| 60 | Finance |
| 70 | Logistics |
| 80 | Customer Care |
| 90 | Guest |
| 100 | Administration |
| 110 | ICT |
| 120 | Server Room |

This provides logical separation between departments while allowing communication through Layer 3 routing.

---

# IPv4 Addressing and Subnetting

The departmental network was created from the `192.168.10.0` address space.

Each department was allocated a `/26` subnet.

```text
Subnet Mask: 255.255.255.192
Prefix:      /26
Addresses:   64 per subnet
Usable:      62 host addresses
```

A `/26` provides enough addressing capacity for approximately 60 devices in each departmental network.

## Department Addressing Table

| VLAN | Department | Network | Gateway | Usable Host Range | Broadcast |
|---:|---|---|---|---|---|
| 10 | Management | `192.168.10.0/26` | `192.168.10.1` | `192.168.10.1 - 192.168.10.62` | `192.168.10.63` |
| 20 | Research | `192.168.10.64/26` | `192.168.10.65` | `192.168.10.65 - 192.168.10.126` | `192.168.10.127` |
| 30 | Human Resources | `192.168.10.128/26` | `192.168.10.129` | `192.168.10.129 - 192.168.10.190` | `192.168.10.191` |
| 40 | Marketing | `192.168.10.192/26` | `192.168.10.193` | `192.168.10.193 - 192.168.10.254` | `192.168.10.255` |
| 50 | Accounting | `192.168.11.0/26` | `192.168.11.1` | `192.168.11.1 - 192.168.11.62` | `192.168.11.63` |
| 60 | Finance | `192.168.11.64/26` | `192.168.11.65` | `192.168.11.65 - 192.168.11.126` | `192.168.11.127` |
| 70 | Logistics | `192.168.11.128/26` | `192.168.11.129` | `192.168.11.129 - 192.168.11.190` | `192.168.11.191` |
| 80 | Customer Care | `192.168.11.192/26` | `192.168.11.193` | `192.168.11.193 - 192.168.11.254` | `192.168.11.255` |
| 90 | Guest | `192.168.12.0/26` | `192.168.12.1` | `192.168.12.1 - 192.168.12.62` | `192.168.12.63` |
| 100 | Administration | `192.168.12.64/26` | `192.168.12.65` | `192.168.12.65 - 192.168.12.126` | `192.168.12.127` |
| 110 | ICT | `192.168.12.128/26` | `192.168.12.129` | `192.168.12.129 - 192.168.12.190` | `192.168.12.191` |
| 120 | Server Room | `192.168.12.192/26` | `192.168.12.193` | `192.168.12.193 - 192.168.12.254` | `192.168.12.255` |

---

# Core Transit Networks

Point-to-point connections between routers and multilayer switches use `/30` networks.

```text
Subnet Mask: 255.255.255.252
Prefix: /30
Usable addresses per subnet: 2
```

The topology uses the following transit networks:

```text
10.10.10.0/30
10.10.10.4/30
10.10.10.8/30
10.10.10.12/30
10.10.10.16/30
10.10.10.20/30
10.10.10.24/30
10.10.10.28/30
10.10.10.32/30
10.10.10.36/30
10.10.10.40/30
10.10.10.44/30
10.10.10.48/30
10.10.10.52/30
```

Using `/30` addressing conserves IPv4 addresses because each point-to-point link only requires two usable addresses.

---

# Technologies Implemented

- Cisco Packet Tracer
- Hierarchical network design
- VLAN segmentation
- IEEE 802.1Q trunking
- IPv4 subnetting
- Layer 3 switching
- Switch Virtual Interfaces
- Inter-VLAN routing
- Centralized DHCP
- DHCP relay
- OSPF
- SSH
- Port security
- Sticky MAC learning
- WPA2 wireless networking
- HTTP/HTTPS services
- Email services
- Redundant network connections
- Spanning Tree Protocol
- Dynamic host configuration
- Connectivity verification

---

# Configuration Walkthrough

## 1. Basic Device Configuration

I first configured the routers and switches with hostnames, passwords, banner messages, password encryption, and disabled DNS lookup.

Example:

```cisco
enable
configure terminal

hostname F1-MGMT-SW

no ip domain-lookup

enable password cisco

service password-encryption

banner motd #AUTHORIZED USERS ONLY#

line console 0
 password cisco
 login
exit

line vty 0 15
 password cisco
 login
exit

end
write memory
```

Each production device was assigned an appropriate hostname instead of using the default Cisco hostname.

Examples include:

```text
F1-MGMT-SW
F1-RESEARCH-SW
F1-HR-SW

F2-MARKETING-SW
F2-ACCOUNTING-SW
F2-FINANCE-SW

F3-LOGISTICS-SW
F3-CUSTOMER-SW
F3-GUEST-SW

F4-ADMIN-SW
F4-ICT-SW
F4-SERVER-SW
```

---

# 2. VLAN Configuration

Each access switch was assigned the VLAN belonging to its department.

Example for the Management switch:

```cisco
enable
configure terminal

vlan 10
 name MANAGEMENT
exit
```

Research:

```cisco
vlan 20
 name RESEARCH
```

Human Resources:

```cisco
vlan 30
 name HUMAN_RESOURCES
```

The same process was repeated for VLANs `40-120`.

---

# 3. Configure Trunk Ports

The first two FastEthernet interfaces on the departmental switches were used as uplinks toward the distribution layer.

```cisco
interface range fa0/1-2
 switchport mode trunk
exit
```

The trunk links allow VLAN traffic to travel between the access and distribution layers.

Verification:

```cisco
show interfaces trunk
```

---

# 4. Configure Access Ports

The remaining switch ports were assigned to the department VLAN.

Example for Management VLAN 10:

```cisco
interface range fa0/3-24
 switchport mode access
 switchport access vlan 10
exit
```

For Research:

```cisco
interface range fa0/3-24
 switchport mode access
 switchport access vlan 20
exit
```

Each departmental switch uses its assigned VLAN number.

Verification:

```cisco
show vlan brief
```

---

# 5. Port Security

Port security was enabled on access interfaces to restrict unauthorized devices.

The configuration:

- Enables port security.
- Allows a maximum of two learned MAC addresses.
- Dynamically learns MAC addresses using sticky learning.
- Places the interface into shutdown state after a violation.

```cisco
interface range fa0/3-24

switchport port-security
switchport port-security maximum 2
switchport port-security mac-address sticky
switchport port-security violation shutdown

exit
```

Verification:

```cisco
show port-security
```

A specific interface can also be checked:

```cisco
show port-security interface fa0/3
```

---

# 6. Distribution Layer Trunks

The downlinks on the multilayer switches were configured as trunks.

Example:

```cisco
interface range gigabitEthernet1/0/3-8
 switchport mode trunk
exit
```

These trunks carry departmental VLANs between the access and distribution layers.

---

# 7. Routed Distribution Uplinks

Links between the multilayer switches and routers operate as Layer 3 interfaces.

A switch interface is converted from Layer 2 to Layer 3 using:

```cisco
interface gigabitEthernet1/0/1
 no switchport
 ip address 10.10.10.1 255.255.255.252
 no shutdown
exit
```

A second routed uplink can be configured using another `/30` subnet:

```cisco
interface gigabitEthernet1/0/2
 no switchport
 ip address 10.10.10.9 255.255.255.252
 no shutdown
exit
```

The exact interface addresses depend on the `/30` link shown in the topology.

---

# 8. Enable Layer 3 Routing

Routing must be enabled on each multilayer switch.

```cisco
configure terminal
ip routing
```

Without `ip routing`, the multilayer switch will not route traffic between VLAN interfaces.

---

# 9. Inter-VLAN Routing

Switch Virtual Interfaces were created to act as the default gateways for each VLAN.

## VLAN 10

```cisco
interface vlan 10
 ip address 192.168.10.1 255.255.255.192
 no shutdown
exit
```

## VLAN 20

```cisco
interface vlan 20
 ip address 192.168.10.65 255.255.255.192
 no shutdown
exit
```

## VLAN 30

```cisco
interface vlan 30
 ip address 192.168.10.129 255.255.255.192
 no shutdown
exit
```

The same process was repeated using the first usable IP address from every departmental subnet.

> When using redundant multilayer switches, duplicate SVI gateway addresses should not be configured simultaneously unless a first-hop redundancy protocol such as HSRP or VRRP is also being used.

---

# 10. DHCP Relay

The DHCP server is located in VLAN 120, while DHCP clients are located across the other VLANs.

Because DHCP discovery uses broadcasts, an `ip helper-address` was configured on each client VLAN SVI.

The DHCP server address used in this implementation is:

```text
192.168.12.196
```

Example:

```cisco
interface vlan 10
 ip address 192.168.10.1 255.255.255.192
 ip helper-address 192.168.12.196
 no shutdown
exit
```

VLAN 20:

```cisco
interface vlan 20
 ip address 192.168.10.65 255.255.255.192
 ip helper-address 192.168.12.196
 no shutdown
exit
```

This was repeated for all DHCP-enabled departmental VLANs.

The server VLAN does not require DHCP relay because its servers use static addressing.

---

# 11. Server Room Configuration

The server room is located on:

```text
VLAN:       120
Network:    192.168.12.192/26
Gateway:    192.168.12.193
```

I used static addressing for the server infrastructure.

Like so:
<img width="939" height="356" alt="image" src="https://github.com/user-attachments/assets/f70d4c35-4517-4874-bdbd-895f3b86ed7f" />


Example layout:

| Device | IPv4 Address |
|---|---|
| VLAN 120 Gateway | `192.168.12.193` |
| Email Server | `192.168.12.194` |
| HTTP/HTTPS Server | `192.168.12.195` |
| DHCP Server | `192.168.12.196` |

All servers use:

```text
Subnet Mask: 255.255.255.192
Default Gateway: 192.168.12.193
```

---

# 12. DHCP Server Configuration

The dedicated DHCP server provides IPv4 addresses to hosts throughout the enterprise network.

In Packet Tracer:

```text
DHCP Server
→ Services
→ DHCP
→ Service: On
```

A separate DHCP pool was created for every client VLAN.

## DHCP Pool Examples

### Management

```text
Pool Name: MGMT
Default Gateway: 192.168.10.1
Starting Address: 192.168.10.6
Subnet Mask: 255.255.255.192
```

### Research

```text
Pool Name: RESEARCH
Default Gateway: 192.168.10.65
Starting Address: 192.168.10.70
Subnet Mask: 255.255.255.192
```

### Human Resources

```text
Pool Name: HR
Default Gateway: 192.168.10.129
Starting Address: 192.168.10.134
Subnet Mask: 255.255.255.192
```

And you can see this in the image here:

<img width="940" height="804" alt="image" src="https://github.com/user-attachments/assets/6513be68-d648-46e0-843f-2533f970b2ee" />


Additional pools were created for:

```text
MARKETING
ACCOUNTING
FINANCE
LOGISTICS
CUSTOMER
GUEST
ADMIN
ICT
```

VLAN 120 was excluded because the servers use static IPv4 addresses.

---

# 13. OSPF Dynamic Routing

OSPF was used to dynamically advertise routes between routers and multilayer switches.

The routers were configured using OSPF process ID `10` and Area `0`.

Example:

```cisco
configure terminal

router ospf 10

network 10.10.10.0 0.0.0.63 area 0

end
write memory
```

The `10.10.10.x` networks represent the routed `/30` infrastructure links.

---

## OSPF on Multilayer Switches

First, Layer 3 routing was enabled:

```cisco
ip routing
```

Then OSPF was configured:

```cisco
router ospf 10

network 10.10.10.0 0.0.0.63 area 0
network 192.168.0.0 0.0.255.255 area 0

exit
```

Because OSPF only activates on locally configured interfaces that match the network statement, each multilayer switch advertises its directly connected VLANs and routed uplinks.

Verification:

```cisco
show ip ospf neighbor
```

```cisco
show ip route
```

```cisco
show ip route ospf
```

---

# 14. Router Interface Configuration

Router interfaces were assigned IP addresses from the appropriate `/30` transit networks.

Example:

```cisco
interface gigabitEthernet0/1

ip address 10.10.10.2 255.255.255.252
no shutdown

exit
```

Example serial interface:

```cisco
interface serial0/2/0

ip address 10.10.10.17 255.255.255.252
no shutdown

exit
```

---

# 15. Serial DCE Clock Rate

Serial interfaces operating as the DCE side require a clock rate.

Example:

```cisco
interface serial0/2/0

clock rate 64000

no shutdown
exit
```

The DCE side can be identified in Packet Tracer by checking the serial connection or using the interface information.

---

# 16. SSH Configuration

SSH was configured on the routers and multilayer switches to provide encrypted remote management.

Example:

```cisco
configure terminal

hostname FLOOR-1-ROUTER

ip domain-name enterprise.local

username netadmin privilege 15 secret Cisco123

crypto key generate rsa
```

When prompted:

```text
1024
```

Enable SSH version 2:

```cisco
ip ssh version 2
```

Configure the VTY lines:

```cisco
line vty 0 15

login local
transport input ssh

exit
```

Save:

```cisco
end
write memory
```

Remote login can then be tested with:

```text
ssh -l netadmin <device-ip-address>
```

---

# 17. Wireless Network Configuration

Each user department contains a wireless access point connected to the department access switch.

Examples of SSIDs include:

```text
MGMT-WIFI
RESEARCH-WIFI
HR-WIFI
MARKETING-WIFI
ACCOUNTING-WIFI
FINANCE-WIFI
LOGISTICS-WIFI
CUSTOMER-WIFI
GUEST-WIFI
ADMIN-WIFI
ICT-WIFI
```



For each access point:

```text
Access Point
→ Config
→ Port 1
```

Example:

```text
SSID: FINANCE-WIFI
Authentication: WPA2-PSK
PSK: <WPA2-PASSWORD>
```

As seen in this case below.

<img width="793" height="362" alt="image" src="https://github.com/user-attachments/assets/1d1121d9-5eaa-44f7-b63b-656f02502077" />


A wireless client can then select the SSID and enter the WPA2 key.

The client is configured to use DHCP so it receives addressing information from the centralized DHCP server.

---

# 18. HTTP/HTTPS Server

The web server was placed inside VLAN 120.

Example configuration:

```text
IPv4 Address: 192.168.12.195
Subnet Mask: 255.255.255.192
Default Gateway: 192.168.12.193
```

In Packet Tracer:

```text
Server
→ Services
→ HTTP
→ HTTP: On
```

HTTPS can also be enabled:

```text
HTTPS: On
```

Testing from a client:

```text
Desktop
→ Web Browser
```

Enter:

```text
http://192.168.12.195
```

Successful page loading verifies that the HTTP server is reachable across the routed enterprise network.

---

# 19. Email Server

The email server was also placed inside VLAN 120.

Example:

```text
IPv4 Address: 192.168.12.194
Subnet Mask: 255.255.255.192
Default Gateway: 192.168.12.193
```

Packet Tracer configuration:

```text
Server
→ Services
→ EMAIL
```

Enable:

```text
SMTP: On
POP3: On
```

Example domain:

```text
enterprise.local
```

User accounts can then be created and configured on the client PCs.

This allows email communication between users in different departments.

---

# Verification and Testing

After configuration, I tested each major network service individually.

---

## Verify VLANs

```cisco
show vlan brief
```

Expected result:

- VLANs `10-120` exist.
- Access interfaces belong to the correct VLAN.

---

## Verify Trunks

```cisco
show interfaces trunk
```

Expected result:

- Access-to-distribution uplinks operate as trunks.
- Required VLANs are allowed across the links.

---

## Verify Layer 3 Interfaces

```cisco
show ip interface brief
```

Expected result:

```text
Status: up
Protocol: up
```

for all active routed interfaces and SVIs.

---

## Verify Routing Table

```cisco
show ip route
```

The routing table should contain:

```text
C - Connected
L - Local
O - OSPF
```

OSPF routes confirm that remote departmental networks are being learned dynamically.

---

## Verify OSPF

```cisco
show ip ospf neighbor
```

This confirms that neighboring OSPF routers and multilayer switches have formed adjacencies.

---

# DHCP Verification

On an end device:

```text
Desktop
→ IP Configuration
→ DHCP
```

The client should automatically receive:

```text
IPv4 Address
Subnet Mask
Default Gateway
DNS information
```

From the command prompt:

```text
ipconfig /all
```

---

# Ping Testing

Local gateway test:

```text
ping 192.168.10.1
```

Cross-VLAN test:

```text
ping 192.168.11.70
```

Cross-floor test:

```text
ping 192.168.12.134
```

Server test:

```text
ping 192.168.12.196
```

Successful responses confirm that:

- VLAN configuration works.
- Inter-VLAN routing works.
- OSPF routing works.
- DHCP relay works.
- End-to-end connectivity is available.

---

# Port Security Verification

```cisco
show port-security
```

For an individual interface:

```cisco
show port-security interface fa0/3
```

Expected configuration:

```text
Port Security: Enabled
Maximum MAC Addresses: 2
Violation Mode: Shutdown
Sticky MAC Learning: Enabled
```

---

# SSH Verification

From another network device:

```text
ssh -l netadmin <router-ip>
```

A successful login confirms secure remote management.

---

# Wireless Verification

A wireless laptop was associated with the departmental access point and configured through DHCP.

After receiving an IP address, I tested communication to another VLAN:

```text
ping 192.168.11.70
```

Successful replies verified that wireless clients could access the routed network.

---

# Troubleshooting

During testing, DHCP failed on two departmental VLANs because their VLAN/SVI configuration did not match the intended addressing plan.

I verified the configuration using:

```cisco
show vlan brief
```

```cisco
show running-config
```

```cisco
show ip interface brief
```

I then corrected the VLAN 50 and VLAN 60 configuration and confirmed that Accounting and Finance clients successfully received DHCP addresses.

This demonstrated the importance of checking:

- VLAN IDs
- SVI addresses
- Default gateways
- DHCP pools
- DHCP helper addresses
- Trunk connectivity

before troubleshooting higher-layer services.

---

# Useful Verification Commands

```cisco
show running-config
show startup-config
show vlan brief
show interfaces trunk
show interfaces status
show ip interface brief
show ip route
show ip route ospf
show ip ospf neighbor
show port-security
show port-security interface fa0/3
show mac address-table
```

Save configuration:

```cisco
copy running-config startup-config
```

or:

```cisco
write memory
```

---

# Final Result

The completed network provides communication between all four floors while maintaining logical separation between departments.

The final implementation includes:

- 12 departmental VLANs
- Multiple `/26` departmental networks
- `/30` point-to-point routed links
- Four core routers
- Redundant multilayer distribution switches
- Department-level access switches
- Centralized DHCP
- DHCP relay
- Inter-VLAN routing
- OSPF
- SSH remote management
- Sticky MAC port security
- Wireless departmental networks
- HTTP/HTTPS services
- Email services
- Dynamic IPv4 assignment
- End-to-end connectivity

The hierarchical architecture provides a structured foundation that can be expanded as additional users, floors, departments, or services are introduced.

---

# Skills Demonstrated

This project demonstrates practical experience with:

```text
Cisco IOS
Cisco Packet Tracer
Enterprise Network Design
Hierarchical Network Architecture
VLAN Configuration
802.1Q Trunking
IPv4 Subnetting
Layer 3 Switching
Inter-VLAN Routing
SVIs
DHCP
DHCP Relay
OSPF
SSH
Port Security
Wireless Networking
Server Configuration
Network Troubleshooting
Network Verification
```

---

## Summary

I designed and implemented a four-floor enterprise network using a hierarchical core, distribution, and access architecture. The project combines VLAN segmentation, `/26` departmental subnetting, inter-VLAN routing, centralized DHCP, DHCP relay, OSPF, SSH, port security, wireless networking, redundant connectivity, and enterprise server services to provide secure communication between all departments.
