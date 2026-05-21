# Cyber Lab

##  Project Description
This project focuses on building a virtual cybersecurity lab environment using multiple virtual machines inside Proxmox.
The lab includes network segmentation, firewall configuration, and traffic control using VyOS.

##  Network Topology

![Network Topology](topology.png)

![Network Interface Bridge](proxmoxbridge.png)
 
## ⚙ VyOS Configuration
This section will document commands used for setting up vyos routers :

### Inteface setup

| VM Name     | IP Address (WAN)     | Lan IP
|-------------|----------------|-------------|
| Vyos0  | 172.18.49.x   | 192.168.10.1/24 , 192.168.20.1/24 |
| Vyos1      | 192.168.10.x (dhcp)  | 10.10.1.1/24     |
| Vyos2       | 192.168.29.x (dhcp)  | 10.10.2.1/24   |
| Vyos3   | 172.18.49.x | 10.10.3.1/24 |

#### Vyos0 
```
configure

set interfaces ethernet eth0 address 172.18.49.X/24
set interfaces ethernet eth1 address 192.168.10.1/24
set interfaces ethernet eth2 address 192.168.20.1/24

commit
save
```
#### DMZ lan network dhcp
```
configure

set service dhcp-server shared-network-name DMZ subnet 192.168.10.0/24 default-router 192.168.10.1
set service dhcp-server shared-network-name DMZ subnet 192.168.10.0/24 range 0 start 192.168.10.10
set service dhcp-server shared-network-name DMZ subnet 192.168.10.0/24 range 0 stop 192.168.19.250

commit
save

```

#### lan network dhcp
```
configure

set service dhcp-server shared-network-name lan subnet 192.168.20.0/24 default-router 192.168.20.0
set service dhcp-server shared-network-name lan subnet 192.168.20.0/24 range 0 start 192.168.20.10
set service dhcp-server shared-network-name lan subnet 192.168.20.0/24 range 0 stop 192.168.20.250

commit
save

```
#### NAT Configuration (Internet Access)
```
configure

set nat source rule 10 outbound-interface name eth0
set nat source rule 10 source address 192.168.0.0/16
set nat source rule 10 translation address masquerade

commit
save

```
#### Dhcp-server and dns (192.168.10.0/24)
```
configure

set service dhcp-server shared-network-name DMZ subnet 192.168.10.0/24 subnet-id 1
set service dhcp-server shared-network-name DMZ subnet 192.168.10.0/24 option default-router 192.168.10.1
set service dhcp-server shared-network-name DMZ subnet 192.168.10.0/24 option name-server 8.8.8.8
set service dhcp-server shared-network-name DMZ subnet 192.168.10.0/24 range 0 start 192.168.10.10
set service dhcp-server shared-network-name DMZ subnet 192.168.10.0/24 range 0 stop 192.168.10.250

commit
save
```

#### Dhcp-server and dns (192.168.20.0/24)
```
configure

set service dhcp-server shared-network-name lan subnet 192.168.20.0/24 subnet-id 2
set service dhcp-server shared-network-name lan subnet 192.168.20.0/24 option default-router 192.168.20.1
set service dhcp-server shared-network-name lan subnet 192.168.20.0/24 option name-server 8.8.8.8
set service dhcp-server shared-network-name lan subnet 192.168.20.0/24 range 0 start 192.168.20.10
set service dhcp-server shared-network-name lan subnet 192.168.20.0/24 range 0 stop 192.168.20.250

commit
save
```

#### Vyos1
```
configure

set interfaces ethernet eth0 address dhcp
set interfaces ethernet eth1 address 10.10.1.1/24

set service dhcp-server shared-network-name lan subnet 10.10.1.0/24 subnet-id 1
set service dhcp-server shared-network-name lan subnet 10.10.1.0/24 option default-router 10.10.1.1
set service dhcp-server shared-network-name lan subnet 10.10.1.0/24 option name-server 8.8.8.8
set service dhcp-server shared-network-name lan subnet 10.10.1.0/24 range 0 start 10.10.1.10
set service dhcp-server shared-network-name lan subnet 10.10.1.0/24 range 0 stop 10.10.1.250

set nat source rule 10 outbound-interface name eth0
set nat source rule 10 source address 10.10.1.0/24
set nat source rule 10 translation address masquerade



commit
save
```
### Vyos2
```
configure

set interfaces ethernet eth0 address dhcp
set interfaces ethernet eth1 address 10.10.2.1/24

set service dhcp-server shared-network-name lan subnet 10.10.2.0/24 subnet-id 1
set service dhcp-server shared-network-name lan subnet 10.10.2.0/24 option default-router 10.10.2.1
set service dhcp-server shared-network-name lan subnet 10.10.2.0/24 option name-server 8.8.8.8
set service dhcp-server shared-network-name lan subnet 10.10.2.0/24 range 0 start 10.10.2.10
set service dhcp-server shared-network-name lan subnet 10.10.2.0/24 range 0 stop 10.10.2.250

set nat source rule 10 outbound-interface name eth0
set nat source rule 10 source address 10.10.2.0/24
set nat source rule 10 translation address masquerade



commit
save
```
### Vyos3
```
configure

set interfaces ethernet eth0 address 172.18.49.x/24
set interfaces ethernet eth1 address 10.10.3.1/24

set service dhcp-server shared-network-name lan subnet 10.10.3.0/24 subnet-id 1
set service dhcp-server shared-network-name lan subnet 10.10.3.0/24 option default-router 10.10.2.1
set service dhcp-server shared-network-name lan subnet 10.10.3.0/24 option name-server 8.8.8.8
set service dhcp-server shared-network-name lan subnet 10.10.3.0/24 range 0 start 10.10.3.10
set service dhcp-server shared-network-name lan subnet 10.10.3.0/24 range 0 stop 10.10.3.250

set nat source rule 10 outbound-interface name eth0
set nat source rule 10 source address 10.10.3.0/24
set nat source rule 10 translation address masquerade



commit
save
```
Network Debugging & Fix Summary

## Overview

This document summarizes the issues encountered and fixes applied to restore connectivity between:

* Attacker network (`10.10.3.0/24`)
* DMZ network (`10.10.1.0/24`)
* Internal network (`10.10.2.0/24`)

The lab runs on **Proxmox** with multiple **VyOS routers**.

---

##  Problems Identified

### 1. Missing Routes on Core Router (vyos0)

vyos0 did not have routes to:

* `10.10.1.0/24` (DMZ)
* `10.10.2.0/24` (Internal)
* `10.10.3.0/24` (Attacker)

Result:

* Traffic defaulted to `172.18.49.1` (university gateway)
* Packets exited the lab environment

---

### 2. Incorrect Default Gateway (Attacker Network)

Misconfigured DHCP option:

```
default-router 10.10.2.1 // this is wrong 
```

Correct:

```
default-router 10.10.3.1 // this is the correct one 
```

Result:

* Clients could not properly route traffic

---

### 3. No Return Path (Asymmetric Routing)

vyos1 (DMZ router) lacked a route back to:

* `10.10.3.0/24`

Result:

* Requests might reach destination
* Responses failed to return

---

### 4. Shared Network with External Infrastructure

Network `172.18.49.0/24` was used for:

* Lab transit traffic
* University network (Proxmox host: `172.18.49.11:8006`)

Result:

* Traffic leakage outside lab
* Unintended routing behavior

---

## ✅ Fixes Applied

### 🔧 1. Static Routes on vyos0 (Core Router)

```
set protocols static route 10.10.1.0/24 next-hop 192.168.10.10
set protocols static route 10.10.2.0/24 next-hop 192.168.20.11
set protocols static route 10.10.3.0/24 next-hop 172.18.49.177
```

---

### 🔧 2. Return Route on vyos1 (DMZ Router)

```
set protocols static route 10.10.3.0/24 next-hop 192.168.10.1
```

---

### 🔧 3. (Optional) Return Route on vyos2

```
set protocols static route 10.10.3.0/24 next-hop 192.168.20.1
```

---

### 🔧 4. Fixed DHCP Gateway (Attacker Network)

```
set service dhcp-server shared-network-name lan subnet 10.10.3.0/24 option default-router 10.10.3.1
```

---

### 🔧 5. Removed NAT for Internal Traffic

* Avoided NAT between internal subnets
* Used routing for inter-network communication

---

## 🧪 Validation

### Connectivity Tests

```
ping 10.10.3.1
ping 172.18.49.1
ping 192.168.10.10
ping 10.10.1.10
```

---

### Traceroute Check

```
traceroute 10.10.1.10
```

Expected:

```
vyos3 → vyos0 → vyos1 → target
```

Incorrect (previous behavior):

```
vyos3 → vyos0 → university gateway → internet
```

---

### Routing Table Verification

```
show ip route
```

Ensure presence of:

* `10.10.1.0/24`
* `10.10.2.0/24`
* `10.10.3.0/24`

---

## 🔁 Final Working Flow

### Forward Path

```
Attacker (10.10.3.x)
   ↓
vyos3
   ↓
vyos0
   ↓
vyos1
   ↓
DMZ Web Server (10.10.1.x)
```

### Return Path

```
Web Server → vyos1 → vyos0 → vyos3 → Attacker
```

---

## ⚠️ Design Limitation

Using `172.18.49.0/24` for both:

* Lab transit
* External network

creates:

* Traffic leakage
* Debugging complexity
* Security risk

---

## 🚀 Recommended Improvement

Create an isolated internal transit network in Proxmox:

```
10.255.255.0/24 (internal-only bridge)
```

Use this for:

* Router interconnections
* Preventing external exposure

---

## 🧠 Key Takeaways

* Define explicit routes between all subnets
* Do not rely on default route for internal traffic
* Ensure bidirectional routing
* Avoid mixing lab and real networks
* Prefer routing over NAT internally

---

## ✅ Status

* Routing fixed
* Connectivity restored
* Traffic contained within lab
* Snort and Wazuh monitoring Implemented(NIDS/NIPS and HIDS)

System is now ready for:

* SOC pipeline integration
* Attack simulation
