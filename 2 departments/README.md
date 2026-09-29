# Cisco Packet Tracer Project: Small network of 2 departments

## Project Overview

This project demonstrates how to design and configure a small departmental network in Cisco Packet Tracer. The goal is to connect two departments—**Accounts** and **Delivery**—using a single router, two access switches, PCs, and printers.

The original case study requires at least two PCs in each department, an appropriate number of routers and switches, correct IP addressing and subnet masks, proper cabling, and successful communication between the two departments.

The starting network is:

```text
192.168.40.0
```

Because there are two departments, the network must be divided into **two subnets**. 

---

## Network Topology

The completed topology contains:

- 1 router
- 2 Cisco 2960 switches
- 4 PCs
- 2 printers
- 2 separate departmental LANs

The Accounts and Delivery departments each connect to their own switch, while both switches connect to Router0. The goal is to specifically build the topology with 1 router, 1 switch per department, 2+ PCs per department, and a printer for each department.

<img width="843" height="556" alt="Screenshot 2026-09-28 214428" src="https://github.com/user-attachments/assets/fcc601d7-d012-4e60-b989-f80dcb70d334" />


From the final Packet Tracer topology:

```text
Accounts Department
Network: 192.168.40.0/25

Delivery Department
Network: 192.168.40.128/25
```

Packet Tracer's automatic connection option was used in the demonstration to select the appropriate cable types between the router, switches, and end devices.

---

# Subnetting

## Step 1: Determine the Number of Required Subnets

There are two departments:

```text
Accounts
Delivery
```

Therefore:

```text
Required subnets = 2
```

The subnet formula used in the project is:

```text
2^n = number of required subnets
```

For two subnets:

```text
2^n = 2

n = 1
```

One bit must therefore be borrowed from the host portion of the original `/24` network.

The new prefix becomes:

```text
/24 + 1 borrowed bit = /25
```

A `/25` prefix corresponds to:

```text
255.255.255.128
```

The block size is:

```text
256 - 128 = 128
```

This creates two subnets.

---

## Subnet 1 — Accounts

```text
Network ID:       192.168.40.0
Subnet Mask:      255.255.255.128
CIDR Prefix:      /25
First Host:       192.168.40.1
Last Host:        192.168.40.126
Broadcast:        192.168.40.127
```

---

## Subnet 2 — Delivery

```text
Network ID:       192.168.40.128
Subnet Mask:      255.255.255.128
CIDR Prefix:      /25
First Host:       192.168.40.129
Last Host:        192.168.40.254
Broadcast:        192.168.40.255
```

The two resulting networks are to be `192.168.40.0/25` and `192.168.40.128/25`.

---

# IP Addressing Plan

| Department | Device | IP Address | Subnet Mask | Default Gateway |
|---|---|---:|---:|---:|
| Accounts | Router0 G0/0 | 192.168.40.1 | 255.255.255.128 | N/A |
| Accounts | PC0 | 192.168.40.2 | 255.255.255.128 | 192.168.40.1 |
| Accounts | PC1 | 192.168.40.3 | 255.255.255.128 | 192.168.40.1 |
| Accounts | Printer0 | 192.168.40.4 | 255.255.255.128 | 192.168.40.1 |
| Delivery | Router0 G0/1 | 192.168.40.129 | 255.255.255.128 | N/A |
| Delivery | PC2 | 192.168.40.130 | 255.255.255.128 | 192.168.40.129 |
| Delivery | PC3 | 192.168.40.131 | 255.255.255.128 | 192.168.40.129 |
| Delivery | Printer1 | 192.168.40.134 | 255.255.255.128 | 192.168.40.129 |

The router interfaces use the first usable address of each subnet and act as the default gateways. The host addressing starts with `.2` in Accounts and `.130` in Delivery because `.1` and `.129` are already assigned to Router0. 

---

# Router Configuration

Router0 performs Layer 3 routing between the Accounts and Delivery networks.

Enter privileged EXEC mode:

```cisco
enable
```

Enter global configuration mode:

```cisco
configure terminal
```

## Enable Both Router Interfaces

Router interfaces are administratively down by default, so they must be enabled.

```cisco
interface range gigabitEthernet 0/0 - 1
no shutdown
exit
```

Then I assigned their addresses. 

---

## Configure the Accounts Interface

```cisco
interface gigabitEthernet 0/0
ip address 192.168.40.1 255.255.255.128
no shutdown
exit
```

This interface connects Router0 to the Accounts switch.

```text
Router0 G0/0
192.168.40.1/25
```

`192.168.40.1` becomes the default gateway for all Accounts devices.

---

## Configure the Delivery Interface

```cisco
interface gigabitEthernet 0/1
ip address 192.168.40.129 255.255.255.128
no shutdown
exit
```

This interface connects Router0 to the Delivery switch.

```text
Router0 G0/1
192.168.40.129/25
```

`192.168.40.129` becomes the default gateway for Delivery devices.

Here I configure `.1` on the first router interface and `.129` on the second interface (since network addresses themselves cannot be assigned as host interface addresses).

---

## Save the Configuration

```cisco
end
write memory
```

Alternatively:

```cisco
copy running-config startup-config
```

---

# Verify Router Configuration

A useful command for checking interface addressing and status is:

```cisco
show ip interface brief
```

Result:

```text
Interface              IP-Address        Status       Protocol
GigabitEthernet0/0     192.168.40.1      up           up
GigabitEthernet0/1     192.168.40.129    up           up
```


The routing table should contain both networks as directly connected:

```text
192.168.40.0/25
192.168.40.128/25
```

And to verify using `show run`:

<img width="892" height="576" alt="Screenshot 2026-09-28 222636" src="https://github.com/user-attachments/assets/6d4a47f0-cbab-4f97-80ce-c07cb66ad28e" />


No static routes are required because both networks are directly attached to Router0.

---

# Configure Accounts Devices

## PC0

Navigate to:

```text
PC0
Desktop
IP Configuration
Static
```

Configure:

```text
IP Address:      192.168.40.2
Subnet Mask:     255.255.255.128
Default Gateway: 192.168.40.1
```

Example:
<img width="999" height="556" alt="Screenshot 2026-09-28 222354" src="https://github.com/user-attachments/assets/af8bc719-7f2c-4c69-a917-153a4cd69149" />


## PC1

```text
IP Address:      192.168.40.3
Subnet Mask:     255.255.255.128
Default Gateway: 192.168.40.1
```

## Printer0

```text
IP Address:      192.168.40.4
Subnet Mask:     255.255.255.128
Default Gateway: 192.168.40.1
```

I assigned these Accounts hosts addresses beginning at `.2`, with `.1` reserved for the router interface.

---

# Configure Delivery Devices

## PC2

```text
IP Address:      192.168.40.130
Subnet Mask:     255.255.255.128
Default Gateway: 192.168.40.129
```

## PC3

```text
IP Address:      192.168.40.131
Subnet Mask:     255.255.255.128
Default Gateway: 192.168.40.129
```

## Printer1

```text
IP Address:      192.168.40.134
Subnet Mask:     255.255.255.128
Default Gateway: 192.168.40.129
```

These addresses remain within the Delivery subnet's usable host range of `192.168.40.129–192.168.40.254`. 

---

# Testing Connectivity

The final requirement is to verify that devices in Accounts can communicate with devices in Delivery.

## Test 1: Accounts to Delivery PC

From an Accounts PC:

```text
ping 192.168.40.130
```

<img width="903" height="581" alt="image" src="https://github.com/user-attachments/assets/04121a88-2edf-4ffc-965c-b264f33fe2cc" />


The first ping can occasionally time out while Packet Tracer performs ARP resolution. Subsequent packets after the first succeeded.

---

## Test 2: Delivery PC to Accounts Printer

From PC3:

```text
ping 192.168.40.4
```

<img width="811" height="621" alt="image" src="https://github.com/user-attachments/assets/1e1463c5-2517-432c-89ed-1d874f87182a" />


So after the cross-subnet ping tests it is clearly confirmed successful communication between hosts and printers in the two departments.

---

# How the Packet Travels Between Departments

Suppose PC0 sends traffic to PC2:

```text
PC0
192.168.40.2
    |
    v
Accounts Switch
    |
    v
Router0 G0/0
192.168.40.1
    |
    | Routing
    v
Router0 G0/1
192.168.40.129
    |
    v
Delivery Switch
    |
    v
PC2
192.168.40.130
```

PC0 determines that `192.168.40.130` is outside its local `/25` subnet. It therefore sends the packet to its default gateway, `192.168.40.1`.

Router0 receives the packet and checks its routing table. Because `192.168.40.128/25` is directly connected through G0/1, Router0 forwards the packet through that interface toward the Delivery switch.

The reverse process occurs when PC2 replies.

---

# Final Network

<img width="843" height="556" alt="Screenshot 2026-09-28 214428" src="https://github.com/user-attachments/assets/2529d6fc-f3da-4a40-b9fb-185884361e1c" />


---

# Takeaways

This project demonstrates several fundamental network-engineering concepts:

- Dividing a `/24` network into two `/25` subnets
- Identifying network, host, and broadcast addresses
- Assigning router interfaces as default gateways
- Configuring static IPv4 addressing on PCs and printers
- Connecting multiple LANs through a router
- Verifying Layer 3 communication using `ping`
- Using `show` commands to troubleshoot Cisco router interfaces

The completed lab satisfies the case-study requirements by building two departmental networks, assigning appropriate addressing, connecting all devices, and successfully testing communication between the Accounts and Delivery subnets. 
