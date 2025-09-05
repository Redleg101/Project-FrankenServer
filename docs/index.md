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
[Phase 1 – Hardware](Phase1-Hardware.md){: .btn .btn-primary }
[Phase 2 – Proxmox](Phase2-Proxmox.md){: .btn .btn-primary }
[Phase 3 – pfSense VM](Phase3-pfSense.md){: .btn .btn-primary }
[Phase 4 – VLANs / Pi-hole / HA](Phase4-VLANs.md){: .btn .btn-primary }
[Diagrams](Diagrams/){: .btn }
[Sanitized Configs (GitHub README)](https://github.com/Redleg101/Project-FrankenServer#configs--templates-sanitized){: .btn }

---

## Addressing (example)

<table>
  <thead>
    <tr>
      <th style="text-align:right;">VLAN</th>
      <th>Name</th>
      <th>Subnet</th>
      <th>Gateway</th>
      <th>Key Hosts</th>
    </tr>
  </thead>
  <tbody>
    <tr><td style="text-align:right;">20</td><td>USERS</td><td><code>10.100.20.0/24</code></td><td><code>10.100.20.1</code></td><td>—</td></tr>
    <tr><td style="text-align:right;">30</td><td>XBOX</td><td><code>10.100.30.0/24</code></td><td><code>10.100.30.1</code></td><td>—</td></tr>
    <tr><td style="text-align:right;">40</td><td>IOT</td><td><code>10.100.40.0/24</code></td><td><code>10.100.40.1</code></td><td>—</td></tr>
    <tr><td style="text-align:right;">60</td><td>SERVICES</td><td><code>10.100.60.0/24</code></td><td><code>10.100.60.1</code></td><td>Printer <code>10.100.60.10</code>, Pi-hole <code>10.100.60.20</code></td></tr>
    <tr><td style="text-align:right;">99</td><td>MGMT</td><td><code>10.100.99.0/24</code></td><td><code>10.100.99.1</code></td><td>Switch/AP/pfSense</td></tr>
  </tbody>
</table>
