# Project FrankenServer
![Last commit](https://img.shields.io/github/last-commit/Redleg101/Project-FrankenServer)
![License](https://img.shields.io/badge/License-MIT-blue.svg)

<!-- Quick badges & links -->
<p align="left">
  <a href="docs/Phase1-Hardware.md"><img src="https://img.shields.io/badge/Phase%201-Hardware-0ea5e9" alt="Phase 1"></a>
  <a href="docs/Phase2-Proxmox.md"><img src="https://img.shields.io/badge/Phase%202-Proxmox-10b981" alt="Phase 2"></a>
  <a href="docs/Phase3-pfSense.md"><img src="https://img.shields.io/badge/Phase%203-pfSense-f59e0b" alt="Phase 3"></a>
  <a href="docs/Phase4-VLANs.md"><img src="https://img.shields.io/badge/Phase%204-VLANs%2FPi--hole%2FHA-8b5cf6" alt="Phase 4"></a>
  <a href="docs/Diagrams/"><img src="https://img.shields.io/badge/View-Diagrams-6366f1" alt="Diagrams"></a>
  <a href="#configs--templates-sanitized"><img src="https://img.shields.io/badge/Configs-Sanitized-64748b" alt="Configs"></a>
</p>

[Open Phase 1](docs/Phase1-Hardware.md) • [Phase 2](docs/Phase2-Proxmox.md) • [Phase 3](docs/Phase3-pfSense.md) • [Phase 4](docs/Phase4-VLANs.md) • [Diagrams](docs/Diagrams/) • [Configs](#configs--templates-sanitized)






> **Note:** Addresses below are examples for documentation. Production uses different ranges and hostnames.

| VLAN | Name | Subnet | Gateway | Key Hosts |
|-----:|------|--------|---------|-----------|
| 20 | USERS | 10.100.20.0/24 | 10.100.20.1 | — |
| 30 | XBOX | 10.100.30.0/24 | 10.100.30.1 | — |
| 40 | IOT | 10.100.40.0/24 | 10.100.40.1 | — |
| 60 | SERVICES | 10.100.60.0/24 | 10.100.60.1 | Printer 10.100.60.10, Pi-hole 10.100.60.20 |
| 99 | MGMT | 10.100.99.0/24 | 10.100.99.1 | Switch/AP/pfSense |

## Configs & Templates (Sanitized)

This repo includes **sanitized example configs** to demonstrate structure and approach without exposing production details.

**Files**
- configs/pfsense-config.example.xml — Illustrative pfSense layout (WAN/LAN trunk, VLANs, DHCP, aliases, example printer rule).  
  *Do not import to production.* Copy and adapt in a private file.
- configs/proxmox-network.example.cfg — Example /etc/network/interfaces for Proxmox (bridges, VLAN-aware trunk).  
  Replace code values like \<WAN-NIC>\, \<LAN-NIC>\, IPs, and gateways before use.

**How to use**
1. Copy an example to a **private** file (e.g., configs/pfsense-config.private.xml) outside of Git history.
2. Replace NIC names, subnets, hostnames, and any credentials/secrets.
3. Validate in a lab first, then apply to production.
4. Never commit real secrets — this repo uses hooks/ignore rules to help prevent that.

**Security note:** All IPs/hostnames shown in docs are examples; production uses different ranges, hostnames, and credentials.

## 🎥 Videos
- Phase 1 – Hardware overview: _Coming soon_
- Phase 2 – Proxmox install: _Coming soon_
- Phase 3 – pfSense deployment: _Coming soon_
- Phase 4 – VLANs, Pi-hole, Home Assistant: _Coming soon_

## TODO
- [ ] Finish SATA port mapping & ZFS pool
- [ ] Replace dual 10GbE NIC or stay 1GbE
- [ ] Add screenshots to all phases
- [ ] Sanitize and upload example diagrams
- [ ] Add video links

[![Website](https://img.shields.io/badge/Website-Live-brightgreen)](https://Redleg101.github.io/Project-FrankenServer/)

