# 🌐 Enterprise Network Evolution (2012-2026)
*From Physical Silicon to Cloud-Native Networking*

## 📖 The "Accidental Admin" Journey
This repository is a technical timeline of my professional evolution. Between **2012 and 2019**, I built and managed company-owned data centers from the ground up. I wasn't just a cloud consumer; I was the person running the cables, racking the Cisco, Huawei and Dell PowerEdge servers, and configuring the physical silicon across Layers 1 through 7.

This project demonstrates my ability to design, automate, and maintain critical infrastructure using **Infrastructure as Code (Vagrant)** and **Modern Network Operating Systems (VyOS)**, bridging the gap between legacy physical environments and modern 2026 architectural standards.

---

## 🏗️ Repository Architecture
The repository is structured into three distinct labs, each representing a leap in scale, redundancy, and design philosophy.

### [🏥 Lab-01: Small Office Collapsed Core](./lab-01-small-office-collapsed-core/README.md)
* **The Origin (2012 Legacy)**: A recreation of my first "Baptism by Fire" environment—building a medical office network under a one-week deadline.
* **Architecture**: 2-Tier Collapsed Core (Edge + Core combined).
* **Focus**: 802.1Q VLAN tagging, Inter-VLAN routing, and secure service isolation (VoIP/Admin).

### [🏢 Lab-02: Campus Enterprise](./lab-02-campus-enterprise-core-dist-access/README.md)
* **Scaling for Resilience**: Reflects the transition into mission-critical campus environments where hardware failure is not an option.
* **Architecture**: Traditional 3-Tier Hierarchical Model (Core, Distribution, Access).
* **Focus**: BGP Core Mesh, VRRP (First-Hop Redundancy), and LACP for high-availability uplinks.

### [🌐 Lab-03: Modern Data Center Fabric](./lab-03-modern-dc-fabric-spine-leaf/README.md)
* **The 2026 Update**: My current standard for high-density, automated data center blocks.
* **Architecture**: L3 Spine-Leaf Fabric with a full-mesh BGP underlay.
* **Focus**: Eliminating Spanning Tree (STP) using eBGP with ECMP for 100% bandwidth utilization and sub-second failover.

---

## 🚀 Getting Started
This entire repository is optimized for **Vagrant** and **VirtualBox**.

### Prerequisites
*   **Vagrant** (v2.2.0+)
*   **VirtualBox** (v6.1+)
*   **System RAM Recommendations**:
    *   **Lab-01**: 8GB+
    *   **Lab-02**: 12GB+
    *   **Lab-03**: 14GB+ (Optimized for 11-VM Data Center)

### Quick Start
```bash
# Clone the repository
git clone https://github.com/your-username/enterprise-network-evolution.git
cd enterprise-network-evolution

# Navigate to a lab and deploy
cd lab-03-modern-dc-fabric-spine-leaf
vagrant up
```

---

## 🛠️ Portfolio Context
These networking labs serve as the "Underlay" for a broader Platform Engineering ecosystem:
* **Hybrid Cloud**: Linking these local fabrics to AWS via VyOS Site-to-Site VPNs.
* **Modern Workloads**: Deploying AWS EKS nodes on top of the routed Leaf switches.
* **Observability**: Monitoring the BGP mesh via Grafana, Prometheus, and Loki.

---

## 🎓 Contact & Links
*   **LinkedIn**: [Tyrel Orde Fecha](https://www.linkedin.com/in/tyrel-orde-fecha-956b21a3)
*   **Portfolio Website**: Coming Soon!