# Cisco Packet Tracer – VLAN Segmentation, DHCP, Wireless Networking, and Inter-VLAN Routing

## Project Overview

This project implements a small branch-office network using a Cisco 2911 router and Cisco 2960 switch. The network is divided into three departments using VLANs, with each department receiving its own wired and wireless network while still being able to communicate with devices in the other VLANs.

The router provides **Router-on-a-Stick inter-VLAN routing** and also acts as the **DHCP server**, allowing client devices to obtain IPv4 addressing automatically. Each department has its own wireless access point for mobile users. These requirements include three separate VLANs, wireless access, automatic IPv4 addressing, and communication between departments.

---

## Network Topology

<img width="1017" height="594" alt="Screenshot 2026-09-29 231523" src="https://github.com/user-attachments/assets/5e906f1b-cb52-47a2-9a7f-4d81e8e28124" />


The network contains:

- **1 Cisco 2911 Router**
- **1 Cisco 2960-24TT Switch**
- **3 Wireless Access Points**
- **3 Desktop PCs**
- **3 Wireless Laptops**
- **3 Department VLANs**

### Department Structure

| Department | VLAN | Network | Default Gateway | Host Range | Broadcast |
|---|---:|---|---|---|---|
| Admin / IT | 10 | `192.168.1.0/27` | `192.168.1.1` | `192.168.1.1 - 192.168.1.30` | `192.168.1.31` |
| Finance / HR | 20 | `192.168.1.32/27` | `192.168.1.33` | `192.168.1.33 - 192.168.1.62` | `192.168.1.63` |
| Customer Service / Reception | 30 | `192.168.1.64/27` | `192.168.1.65` | `192.168.1.65 - 192.168.1.94` | `192.168.1.95` |

The `/27` subnet mask is:

```text
255.255.255.224
```

Each subnet provides 30 usable host addresses, which is more than enough for this small branch-office topology while maintaining separation between departments.

---

# 1. VLAN Configuration

Three VLANs are created on the Cisco 2960 switch.

```cisco
enable
configure terminal

vlan 10
 name ADMIN_IT
exit

vlan 20
 name FINANCE_HR
exit

vlan 30
 name CUSTOMER_SERVICE
exit
```

VLAN 10 separates Admin/IT devices, VLAN 20 contains Finance/HR devices, and VLAN 30 contains Customer Service/Reception devices.

VLANs create separate Layer 2 broadcast domains. Devices in different VLANs cannot communicate directly without a Layer 3 routing device, which is why inter-VLAN routing is configured later.

---

# 2. Assign Switch Ports to VLANs

The access ports connected to each department are configured as VLAN access ports.

### VLAN 10 – Admin / IT

```cisco
interface range fa0/2-4
 switchport mode access
 switchport access vlan 10
exit
```

### VLAN 20 – Finance / HR

```cisco
interface range fa0/5-7
 switchport mode access
 switchport access vlan 20
exit
```

### VLAN 30 – Customer Service / Reception

```cisco
interface range fa0/8-10
 switchport mode access
 switchport access vlan 30
exit
```

This places each connected PC and wireless access point into its appropriate departmental VLAN. The access-port configuration follows the same department-to-port segmentation implemented on the switch. 

---

# 3. Configure the Switch Trunk

The link between the switch and router must carry traffic from all three VLANs.

Because multiple VLANs share this single physical connection, the switch interface connected to the router is configured as a trunk.

```cisco
interface fa0/1
 switchport mode trunk
exit

end
write memory
```

A trunk differs from an access port because it can transport traffic belonging to multiple VLANs.

The trunk link carries:

```text
VLAN 10
VLAN 20
VLAN 30
```

---

# 4. Verify VLAN Configuration

The VLAN assignments can be verified with:

```cisco
show vlan brief
```

The expected output should show VLANs 10, 20, and 30 along with their assigned interfaces.

The trunk can be checked with:

```cisco
show interfaces trunk
```

---

# 5. Configure Router-on-a-Stick

Since only one physical router interface connects to the switch, **Router-on-a-Stick** is used.

The physical `GigabitEthernet0/0` interface is enabled first.

```cisco
enable
configure terminal

interface gigabitEthernet0/0
 no shutdown
exit
```

Three logical subinterfaces are then created from this physical interface.

Each subinterface represents one VLAN and becomes the default gateway for that VLAN. This approach uses IEEE 802.1Q VLAN tagging.

---

## VLAN 10 Subinterface

```cisco
interface gigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.1.1 255.255.255.224
exit
```

`192.168.1.1` becomes the default gateway for Admin/IT devices.

---

## VLAN 20 Subinterface

```cisco
interface gigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.1.33 255.255.255.224
exit
```

`192.168.1.33` becomes the default gateway for Finance/HR devices.

---

## VLAN 30 Subinterface

```cisco
interface gigabitEthernet0/0.30
 encapsulation dot1Q 30
 ip address 192.168.1.65 255.255.255.224
exit
```

`192.168.1.65` becomes the default gateway for Customer Service/Reception devices.

Save the configuration:

```cisco
end
write memory
```

---

# 6. Verify Router Subinterfaces

Use:

```cisco
show ip interface brief
```

The router should show:

```text
GigabitEthernet0/0       up
GigabitEthernet0/0.10    up
GigabitEthernet0/0.20    up
GigabitEthernet0/0.30    up
```

Each subinterface acts as the Layer 3 gateway for its associated VLAN.

---

# 7. Configure DHCP

The router also provides DHCP services so PCs and laptops do not require manually configured IPv4 addresses.

DHCP service on Cisco routers is enabled with:

```cisco
configure terminal
service dhcp
```

The router-based DHCP configuration allows hosts in each VLAN to receive addresses automatically. 

---

## Reserve Gateway Addresses

A few addresses at the beginning of each subnet can be excluded from DHCP so they remain available for infrastructure devices.

```cisco
ip dhcp excluded-address 192.168.1.1 192.168.1.5
ip dhcp excluded-address 192.168.1.33 192.168.1.37
ip dhcp excluded-address 192.168.1.65 192.168.1.69
```

---

## Admin / IT DHCP Pool

```cisco
ip dhcp pool ADMIN_IT
 network 192.168.1.0 255.255.255.224
 default-router 192.168.1.1
exit
```

---

## Finance / HR DHCP Pool

```cisco
ip dhcp pool FINANCE_HR
 network 192.168.1.32 255.255.255.224
 default-router 192.168.1.33
exit
```

---

## Customer Service / Reception DHCP Pool

```cisco
ip dhcp pool CUSTOMER_SERVICE
 network 192.168.1.64 255.255.255.224
 default-router 192.168.1.65
exit
```

Save the router configuration:

```cisco
end
write memory
```

---

# 8. Configure End Devices for DHCP

Each PC is configured through:

```text
Desktop
→ IP Configuration
→ DHCP
```

After requesting an address, a VLAN 10 device should receive an address from the `192.168.1.0/27` network.

A VLAN 20 device should receive an address from:

```text
192.168.1.32/27
```

A VLAN 30 device should receive an address from:

```text
192.168.1.64/27
```

The DHCP lease should also provide the appropriate VLAN gateway.

DHCP bindings can be checked from the router with:

```cisco
show ip dhcp binding
```

Additional DHCP information can be viewed using:

```cisco
show ip dhcp pool
```

---

# 9. Wireless Network Configuration

Each department has a dedicated wireless access point connected to an access port belonging to that department's VLAN.

Example wireless configuration:

| VLAN | Department | SSID |
|---:|---|---|
| 10 | Admin / IT | `Admin-WiFi` |
| 20 | Finance / HR | `Finance-WiFi` |
| 30 | Customer Service / Reception | `CS-WiFi` |

Each access point can be configured in Packet Tracer under:

```text
Access Point
→ Config
→ Port 1
```

Configure the SSID and select:

```text
WPA2-PSK
```

Like so: <img width="1439" height="834" alt="image" src="https://github.com/user-attachments/assets/e1e1ab74-78ee-45ac-a761-e52d6368188f" />



A unique WPA2 passphrase should be configured for each department.

The network design uses a separate wireless access point for each departmental VLAN. 

---

# 10. Connect the Wireless Laptops

For laptops using a wireless module, the laptop may first need a compatible wireless network adapter installed.

In Packet Tracer:

```text
Laptop
→ Physical
→ Power Off
→ Install WPC300N Wireless Module
→ Power On
```

Then navigate to:

```text
Desktop
→ PC Wireless
→ Connect
```

Select the SSID associated with the laptop's department and enter the WPA2 passphrase.

After connecting, configure IPv4 addressing through DHCP.

```text
Desktop
→ IP Configuration
→ DHCP
```

The wireless client should receive an IP address from the same subnet as the wired devices in that VLAN.

---

# 11. Inter-VLAN Routing

Without the router, VLAN 10, VLAN 20, and VLAN 30 remain isolated.

Router-on-a-Stick changes this by allowing traffic to travel:

```text
PC
   ↓
Cisco 2960 Switch
   ↓
802.1Q Trunk
   ↓
Cisco 2911 Router
   ↓
Router Subinterface
   ↓
Destination VLAN
```

For example, when an Admin/IT PC communicates with a Finance/HR PC, the packet is sent to `192.168.1.1`, routed between the VLAN subinterfaces, and returned through the trunk toward VLAN 20.

---

# 12. Verify Routing

Check the router's routing table:

```cisco
show ip route
```

The router should contain connected routes similar to:

```text
192.168.1.0/27
192.168.1.32/27
192.168.1.64/27
```

Because all three networks are directly connected through router subinterfaces, no static routes or dynamic routing protocol are required.

---

# 13. Connectivity Testing

Testing should first be performed inside each VLAN and then between different VLANs.

From an Admin/IT PC:

```text
ping 192.168.1.1
```

This verifies connectivity to the VLAN 10 gateway.

Then test communication with a Finance/HR host:

```text
ping 192.168.1.x
```

Finally, test communication with a Customer Service/Reception host:

```text
ping 192.168.1.x
```

Successful inter-department pings confirm that VLAN tagging, trunking, subinterfaces, DHCP, and inter-VLAN routing are functioning correctly. Connectivity testing between hosts in different departments is the final verification step of the implementation. 

---

# Useful Verification Commands

### Switch

```cisco
show vlan brief
show interfaces trunk
show interfaces switchport
show running-config
```

### Router

```cisco
show ip interface brief
show ip route
show ip dhcp binding
show ip dhcp pool
show running-config
```

---

# Complete Switch Configuration

```cisco
enable
configure terminal

vlan 10
 name ADMIN_IT
exit

vlan 20
 name FINANCE_HR
exit

vlan 30
 name CUSTOMER_SERVICE
exit

interface range fa0/2-4
 switchport mode access
 switchport access vlan 10
exit

interface range fa0/5-7
 switchport mode access
 switchport access vlan 20
exit

interface range fa0/8-10
 switchport mode access
 switchport access vlan 30
exit

interface fa0/1
 switchport mode trunk
exit

end
write memory
```

# Complete Router Configuration

```cisco
enable
configure terminal

interface gigabitEthernet0/0
 no shutdown
exit

interface gigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.1.1 255.255.255.224
exit

interface gigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.1.33 255.255.255.224
exit

interface gigabitEthernet0/0.30
 encapsulation dot1Q 30
 ip address 192.168.1.65 255.255.255.224
exit

service dhcp

ip dhcp excluded-address 192.168.1.1 192.168.1.5
ip dhcp excluded-address 192.168.1.33 192.168.1.37
ip dhcp excluded-address 192.168.1.65 192.168.1.69

ip dhcp pool ADMIN_IT
 network 192.168.1.0 255.255.255.224
 default-router 192.168.1.1
exit

ip dhcp pool FINANCE_HR
 network 192.168.1.32 255.255.255.224
 default-router 192.168.1.33
exit

ip dhcp pool CUSTOMER_SERVICE
 network 192.168.1.64 255.255.255.224
 default-router 192.168.1.65
exit

end
write memory
```

## Project Result

The completed Packet Tracer network separates Admin/IT, Finance/HR, and Customer Service/Reception into independent VLANs while maintaining controlled Layer 3 communication through the Cisco router. DHCP eliminates manual client addressing, 802.1Q trunking transports all VLANs across a single router connection, and departmental wireless access points extend each VLAN to wireless clients.

This project demonstrates practical experience with **IPv4 subnetting, VLAN creation, access-port assignment, 802.1Q trunking, Router-on-a-Stick, DHCP, wireless LAN configuration, default gateways, and network connectivity verification**.
