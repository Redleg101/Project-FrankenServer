# Project FrankenServer





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
