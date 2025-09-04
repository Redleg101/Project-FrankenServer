---
title: Home
nav_order: 1
---

# Project FrankenServer
{: .fs-9 }

Proxmox + pfSense homelab showcasing virtualization, VLAN segmentation, Pi-hole, and Home Assistant.

> **Note:** IPs/hostnames shown here are sanitized. Production uses different ranges, hostnames, and credentials.
{: .note }

### Quick links
<a class=\"btn btn-primary\" href=\"../docs/Phase1-Hardware.md\">Phase 1 – Hardware</a>
<a class=\"btn btn-primary\" href=\"../docs/Phase2-Proxmox.md\">Phase 2 – Proxmox</a>
<a class=\"btn btn-primary\" href=\"../docs/Phase3-pfSense.md\">Phase 3 – pfSense VM</a>
<a class=\"btn btn-primary\" href=\"../docs/Phase4-VLANs.md\">Phase 4 – VLANs / Pi-hole / HA</a>
<a class=\"btn\" href=\"../docs/Diagrams/\">Diagrams</a>
<a class=\"btn\" href=\"../#configs--templates-sanitized\">Sanitized Configs</a>

---

## Addressing (example)
| VLAN | Name | Subnet | Gateway | Key Hosts |
|-----:|------|--------|---------|-----------|
| 20 | USERS | 10.100.20.0/24 | 10.100.20.1 | — |
| 30 | XBOX | 10.100.30.0/24 | 10.100.30.1 | — |
| 40 | IOT | 10.100.40.0/24 | 10.100.40.1 | — |
| 60 | SERVICES | 10.100.60.0/24 | 10.100.60.1 | Printer 10.100.60.10, Pi-hole 10.100.60.20 |
| 99 | MGMT | 10.100.99.0/24 | 10.100.99.1 | Switch/AP/pfSense |
