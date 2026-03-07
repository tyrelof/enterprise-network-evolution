# Legacy-to-Modern: Layer 3 Spine-Leaf Fabric with VyOS

## 1. VirtualBox Setup: Internal Networks (Virtual Cables)
To create physical isolation and point-to-point links between your Spines and Leafs, configure the following "Internal Network" names for each VM's Network Adapter.

| Link | Source Interface | Destination Interface | Internal Network Name |
| :--- | :--- | :--- | :--- |
| **Spine1 <-> Leaf1** | Spine1 - eth1 | Leaf1 - eth1 | `s1-l1-link` |
| **Spine1 <-> Leaf2** | Spine1 - eth2 | Leaf2 - eth1 | `s1-l2-link` |
| **Spine1 <-> Leaf3** | Spine1 - eth3 | Leaf3 - eth1 | `s1-l3-link` |
| **Spine1 <-> Leaf4** | Spine1 - eth4 | Leaf4 - eth1 | `s1-l4-link` |
| **Spine2 <-> Leaf1** | Spine2 - eth1 | Leaf1 - eth2 | `s2-l1-link` |
| **Spine2 <-> Leaf2** | Spine2 - eth2 | Leaf2 - eth2 | `s2-l2-link` |
| **Spine2 <-> Leaf3** | Spine2 - eth3 | Leaf3 - eth2 | `s2-l3-link` |
| **Spine2 <-> Leaf4** | Spine2 - eth4 | Leaf4 - eth2 | `s2-l4-link` |
| **Leaf1 <-> EKS**    | Leaf1 - eth3 | server-eks  | `leaf1-vlan10` |
| **Leaf2 <-> Stateful**| Leaf2 - eth3 | server-stateful| `leaf2-vlan10` |
| **Leaf3 <-> Storage** | Leaf3 - eth3 | server-storage | `leaf3-vlan10` |
| **Leaf4 <-> Firewall**| Leaf4 - eth3 | server-firewall| `leaf4-vlan10` |

---

## 2. Modernization Logic: Why are we killing STP?
In legacy 3-Tier networks, Spanning Tree Protocol (STP) blocked redundant links to prevent broadcast storms. This meant that if you bought two 10Gbps uplinks, one was always disabled—leaving you with an inefficient active/passive topology.

### The Modern Way (Layer 3 & ECMP)
By pushing Layer 3 routing directly down to the Leaf switches and completely removing Layer 2 from the core links, we eliminate the need for STP. BGP treats every link as an active, routed path.
* **ECMP (Equal-Cost Multi-Pathing)** load-balances traffic evenly across all Spine switches. If you have 2 Spines, you get 2 active paths. 100% bandwidth utilization.
* **Fast Failover**: If a cable is cut ("East-West" traffic failure), BGP immediately withdraws the route for the dead link. Traffic instantly hashes over the remaining healthy Spines. No STP blocking/listening/learning states to wait for.

---

## 3. The Gateway Strategy (Leafs)
In this lab, the Leaf switches act as the **Layer 3 Default Gateway** for the respective server segments.
By terminating the VLANs locally at the rack level:
* **Leaf1** handles **EKS Workers** (Subnet: `10.10.10.0/24`, Gateway: `10.10.10.1`).
* **Leaf2** handles **Stateful Servers** (Subnet: `10.10.20.0/24`, Gateway: `10.10.20.1`).
* **Leaf3** handles **Storage Array** (Subnet: `10.10.30.0/24`, Gateway: `10.10.30.1`).
* **Leaf4** handles **Firewall Services** (Subnet: `10.10.40.0/24`, Gateway: `10.10.40.1`).

*Note: In advanced setups, EVPN/VXLAN is used to stretch the same subnet across Leafs. For standard L3, routing unique subnets at each Leaf is best practice.*

---

## 4. Configuration: Copy / Paste
*Note: The BGP syntax below is optimized for modern **VyOS 1.4+ (Sagitta)** which uses `system-as`. If using VyOS 1.3, replace `set protocols bgp system-as 65001` with `set protocols bgp 65001`, and `address-family` commands accordingly.*

### Spine 1 (AS 65001)
```bash
configure

# Loopback
set interfaces loopback lo address 10.255.0.1/32

# P2P Links to Leafs
set interfaces ethernet eth1 address 10.0.1.1/30
set interfaces ethernet eth2 address 10.0.1.5/30
set interfaces ethernet eth3 address 10.0.1.9/30
set interfaces ethernet eth4 address 10.0.1.13/30

# BGP Configuration
set protocols bgp system-as 65001
set protocols bgp parameters router-id 10.255.0.1
set protocols bgp parameters default multipath

# Peering to Leafs
set protocols bgp neighbor 10.0.1.2 remote-as 65101  # LEAF1
set protocols bgp neighbor 10.0.1.6 remote-as 65102  # LEAF2
set protocols bgp neighbor 10.0.1.10 remote-as 65103 # LEAF3
set protocols bgp neighbor 10.0.1.14 remote-as 65104 # LEAF4

set protocols bgp neighbor 10.0.1.2 address-family ipv4-unicast
set protocols bgp neighbor 10.0.1.6 address-family ipv4-unicast
set protocols bgp neighbor 10.0.1.10 address-family ipv4-unicast
set protocols bgp neighbor 10.0.1.14 address-family ipv4-unicast

# Advertise Loopback
set protocols bgp address-family ipv4-unicast network 10.255.0.1/32

commit
save
```

### Spine 2 (AS 65001)
```bash
configure

# Loopback
set interfaces loopback lo address 10.255.0.2/32

# P2P Links to Leafs
set interfaces ethernet eth1 address 10.0.2.1/30
set interfaces ethernet eth2 address 10.0.2.5/30
set interfaces ethernet eth3 address 10.0.2.9/30
set interfaces ethernet eth4 address 10.0.2.13/30

# BGP Configuration
set protocols bgp system-as 65001
set protocols bgp parameters router-id 10.255.0.2
set protocols bgp parameters default multipath

# Peering to Leafs
set protocols bgp neighbor 10.0.2.2 remote-as 65101  # LEAF1
set protocols bgp neighbor 10.0.2.6 remote-as 65102  # LEAF2
set protocols bgp neighbor 10.0.2.10 remote-as 65103 # LEAF3
set protocols bgp neighbor 10.0.2.14 remote-as 65104 # LEAF4

set protocols bgp neighbor 10.0.2.2 address-family ipv4-unicast
set protocols bgp neighbor 10.0.2.6 address-family ipv4-unicast
set protocols bgp neighbor 10.0.2.10 address-family ipv4-unicast
set protocols bgp neighbor 10.0.2.14 address-family ipv4-unicast

# Advertise Loopback
set protocols bgp address-family ipv4-unicast network 10.255.0.2/32

commit
save
```

### Leaf Switches (AS 65101-65104)
*Example for Leaf 1 (AS 65101). Duplicate and increment IP/AS for others.*
```bash
configure

# Loopback (Use .11 for L1, .12 for L2, .13 for L3, .14 for L4)
set interfaces loopback lo address 10.255.0.11/32

# P2P Links to Spines
set interfaces ethernet eth1 address 10.0.1.2/30  # To Spine 1 (Eth matches Leaf index)
set interfaces ethernet eth2 address 10.0.2.2/30  # To Spine 2

# Server Segment (Gateway)
set interfaces ethernet eth3 vif 10 address 10.10.10.1/24

# BGP
set protocols bgp system-as 65101
set protocols bgp parameters router-id 10.255.0.11
set protocols bgp parameters default multipath

# Peering to Spines
set protocols bgp neighbor 10.0.1.1 remote-as 65001 # SPINE1
set protocols bgp neighbor 10.0.2.1 remote-as 65001 # SPINE2
set protocols bgp neighbor 10.0.1.1 address-family ipv4-unicast
set protocols bgp neighbor 10.0.2.1 address-family ipv4-unicast

# Advertise Subnets
set protocols bgp address-family ipv4-unicast network 10.255.0.11/32
set protocols bgp address-family ipv4-unicast network 10.10.10.0/24

commit
save
```
