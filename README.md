# Faculty of Theft — Infrastructure Systems and Services

> Enterprise-style network infrastructure lab built and simulated in GNS3: dual-firewall high availability (OPNsense + CARP), VLAN segmentation, centralized network services, and Zabbix monitoring.

Project for course **06016420 Infrastructure Systems and Services**, Department of Information Technology, King Mongkut's Institute of Technology Ladkrabang (KMITL) — Semester 2/2568.

---

## 📖 Overview

**Faculty of Theft** (คณะโจรกรรมศาสตร์) is a fictional organization used as the case study for this project. The goal is to design, build, and validate a small-to-medium enterprise network infrastructure — covering topology design, IP addressing, VLAN segmentation, high-availability gateways, core services, and real-time monitoring — entirely simulated inside [GNS3](https://www.gns3.com/).

The report and configuration exports in this repository walk through the full lifecycle: **design → implementation → service deployment → testing → real-world simulation & troubleshooting.**

## 🏗️ Network Topology

The network is organized around three client VLANs and one server zone, all routed and protected by a pair of redundant firewalls.

| Segment | Description | Subnet |
|---|---|---|
| VLAN 10 | สาขาโจรกรรมยานยนต์ (Vehicle Theft Dept.) | `192.168.10.0/24` |
| VLAN 20 | สาขาโจรกรรมข้อมูล (Data Theft Dept.) | `192.168.20.0/24` |
| VLAN 30 | โจรกรรมศาสตร์ หลักสูตรนานาชาติ (International Theft Studies) | `192.168.30.0/24` |
| Server Zone | Centralized servers | `192.168.40.0/24` |

Full topology diagram and per-interface IP assignments are available in [`docs/ISAS-project-report.pdf`](docs/ISAS-project-report.pdf) (Chapter 2).

## 🧰 Tech Stack

| Component | Role |
|---|---|
| **OPNsense** (x2) | Firewall, Routing, NAT, DHCP, DNS — deployed in an HA pair via CARP |
| **Cisco IOSvL2** | Layer 2 switching, VLAN trunking, inter-switch links |
| **Ubuntu Server** | Web Server, File/TFTP Server, Zabbix Server |
| **Ubuntu Desktop** | Client machines across VLANs |
| **Zabbix** | Centralized monitoring and alerting |
| **GNS3** | Full network virtualization/simulation platform |

## ✨ Key Features

- **High Availability Gateway** — dual OPNsense firewalls with CARP (Common Address Redundancy Protocol) for automatic failover
- **Network Segmentation** — VLAN-based isolation between departments with controlled inter-VLAN routing
- **Link Aggregation** — bundled links between core devices for added throughput/redundancy
- **Centralized Services** — DHCP, DNS, Web, and TFTP/File services served from a dedicated server zone
- **System Monitoring** — Zabbix deployment for real-time host and service health tracking

## 📁 Repository Structure

```
faculty-of-theft-infra/
├── docs/                    # Full project report and reference diagrams
├── topology/                # GNS3 project file
├── configs/
│   ├── opnsense/            # Firewall configuration exports
│   └── switches/            # Switch configuration exports
└── testing/
    └── test-results/        # Connectivity & service test evidence
```

## ⚠️ Security Note

The credentials that appear in the files under `configs/` (e.g. `enable secret`, `username cisco password ...`) are **default lab values used for this simulated environment only** — they are not real production credentials and are safe to be public. Do not reuse these credentials in any real-world deployment.

## ✅ Testing Summary

The system was validated across two levels (see Chapter 5 of the full report for details):

- **Network Testing** — intra-VLAN connectivity, inter-VLAN routing, internet access, CARP failover
- **Service Testing** — Web, TFTP, DHCP, DNS, Zabbix monitoring, OPNsense reporting, SSH access

Chapter 6 of the report also covers real-world usage simulation and troubleshooting of issues encountered during deployment.

## 👥 Contributors

| Student ID | Name | Responsibilities |
|---|---|---|
| 67070183 | สาริน สุขสอน | Provisioned virtual resources; installed & configured services; service testing |
| 67070193 | อติกันต์ ชินวรรณโณ | Designed system & topology; provisioned virtual resources; overall system testing; installed & tested monitoring system |

**Advisor:** ผศ. อัครินทร์ คุณกิตติ

## 📄 Full Report

The complete project report (Thai), including detailed configuration steps, IP addressing tables, and testing screenshots, is available at [`docs/ISAS-project-report.pdf`](docs/ISAS-project-report.pdf).

## 📜 License

This project was created for educational purposes as part of coursework at KMITL. Feel free to reference the design for learning purposes.