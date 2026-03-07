🌐 Lab-03: Modern Data Center Fabric (Spine-Leaf)
From Accidental Admin (2012) to Infrastructure Architect (2026)
📖 The Evolution: The 2026 Architectural Update

From 2012 to 2019, I built and managed company-owned data centers with Huawei, Dell PowerEdge, and Proxmox/VMware ESXi—not as a cloud consumer, but as the person running the cables, racking the servers, and configuring the silicon from Layer 1 to Layer 7.

This lab is my **2026 Architectural Update**. I have taken the legacy 3-Tier models I used to build manually and replaced them with a modern **Layer 3 Spine-Leaf Fabric**. This represents my current transition into cloud-aligned networking: killing Spanning Tree (STP) in favor of eBGP with ECMP to create a self-healing, automated fabric.

🏗️ Topology Architecture

This lab simulates a high-density data center block with a full-mesh BGP underlay.

    Spine Layer: 2x VyOS Routers (AS 65001) acting as the high-speed backbone.

    Leaf Layer: 4x VyOS Switches (AS 65101-65104) providing L3 gateways for server racks.

    Edge Layer: 1x VyOS Border Router (AS 65000) providing North-South Internet connectivity.

    Endpoints: 4x Alpine Linux nodes simulating EKS Workers, Stateful Servers, Storage, and Firewalls.

🚀 Quick Start
Bash

# 1. Clone the repo and enter the directory
mkdir lab-03-modern-dc-fabric && cd lab-03-modern-dc-fabric

# 2. Fire up the 11-VM Data Center (Requires Vagrant & VirtualBox)
# Note: This lab is optimized for high-spec machines (14GB+ RAM available)
vagrant up

🛠️ The "SRE" BGP Cheat Sheet

Once the lab is up, use these commands to verify the fabric.
1. Verification (The "Handshake")

Check if your neighbors are active. You should see PfxRcd as a number (e.g., 4).
Bash

# Run on leaf1
vtysh -c "show ip bgp summary"

2. The "STP Killer" (ECMP Check)

Verify that you are using both Spines at once for traffic. You should see two "via" lines for a single destination.
Bash

# Look for the route to the Storage Array (Leaf 3)
vtysh -c "show ip route 10.10.30.0/24"

3. End-to-End Connectivity

Test the path from the EKS nodes to the Storage Array.
Bash

vagrant ssh server-eks
ping 10.10.30.50  # Ping the Storage Array

4. Chaos Engineering (Failover Test)

Prove the resilience of the fabric.
Bash

# Continuous ping from a server
ping 10.10.30.50

# In another terminal, kill a Spine
vagrant halt spine1

# Observation: Notice the sub-second failover as BGP withdraws the route.

🧠 Technical Highlights

    BGP Unnumbered/Peering: Every link is a routed point-to-point /30 subnet.

    L3-Only Core: No STP loops possible. No blocked links.

    VLAN Tagging (802.1Q): Leaf switches handle vif 10 for server isolation.

    Infrastructure as Code: The entire network state is defined in the Vagrantfile.

#