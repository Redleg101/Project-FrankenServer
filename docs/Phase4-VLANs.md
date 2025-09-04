---
title: Phase 4 – VLANs / Pi-hole / HA
nav_order: 40
---
# Phase 4 – VLANs, Pi-hole, and Home Assistant

## 🎯 Objective
Finalize network segmentation, deploy Pi-hole for DNS filtering, and stand up Home Assistant—while keeping inter-VLAN access tightly controlled and your printer reachable from all user nets.

---

## 🧩 VLAN Plan (sanitized examples)
| VLAN | Name        | Subnet         | GW          | Notes                                  |
|-----:|-------------|----------------|-------------|----------------------------------------|
| 10   | AP-MGMT     | <PUBLIC-IP-REDACTED>/24 | <PUBLIC-IP-REDACTED> | Optional AP mgmt                       |
| 20   | USERS       | <PUBLIC-IP-REDACTED>/24 | <PUBLIC-IP-REDACTED> | PCs/laptops                            |
| 30   | XBOX        | <PUBLIC-IP-REDACTED>/24 | <PUBLIC-IP-REDACTED> | UPnP allowed                           |
| 40   | IOT         | <PUBLIC-IP-REDACTED>/24 | <PUBLIC-IP-REDACTED> | Isolation, allow printer only          |
| 60   | SERVICES    | <PUBLIC-IP-REDACTED>/24 | <PUBLIC-IP-REDACTED> | Printer @ <PUBLIC-IP-REDACTED>, Pi-hole @ .20  |
| 99   | MGMT        | <PUBLIC-IP-REDACTED>/24 | <PUBLIC-IP-REDACTED> | Switch/AP/pfSense mgmt                 |

---

## 🖥️ LXC/VM Deployments
### Pi-hole (recommended: Proxmox LXC)
- CPU/RAM: 1 vCPU / 512–1024 MB
- Static IP: **<PUBLIC-IP-REDACTED>/24** (VLAN 60)
- Upstreams: Cloudflare (<PUBLIC-IP-REDACTED>/<PUBLIC-IP-REDACTED>) or Quad9 (<PUBLIC-IP-REDACTED>)
- Blocklists: default + your choice; enable DHCP *only if* pfSense isn’t doing DHCP (here pfSense remains DHCP).

### Home Assistant
- Option A: HAOS VM (easiest)
- Option B: Container (advanced)
- IP: **<PUBLIC-IP-REDACTED>/24** (or IOT if you prefer)
- mDNS discovery across VLANs: enable Avahi/MDNS reflector (see below)

---

## 🔧 pfSense Changes

### 1) DNS Flow Options
**Option 1 (simple):** Clients → pfSense (DNS Resolver) → Pi-hole as upstream  
- Services → DNS Resolver → *Enable forwarding mode* → Upstream: **<PUBLIC-IP-REDACTED>**

**Option 2 (preferred filtering):** Clients → **Pi-hole directly** → (Pi-hole → upstream)  
- DHCP Server → each VLAN → **DNS Server** = <PUBLIC-IP-REDACTED>  
- Keep DNS Resolver in **non-forwarding** (or disable) to avoid bypass.

> Whichever you choose, **block direct external DNS** from clients to the Internet so everything traverses your resolver.
### 2) Firewall Rules (per VLAN)
Order matters—top to bottom:
1. **Allow Essentials to pfSense/Pi-hole**  
   - UDP 53 (DNS), TCP 53 (if needed), UDP 123 (NTP), TCP 80/443 to pfSense GUI only from MGMT.
2. **Block Inter-VLAN by default**  
   - Source: VLAN net → Dest: RFC1918 → **Block**
3. **Allow Printing Exception**  
   - Source: USERS/XBOX/IOT → Dest: **<PUBLIC-IP-REDACTED>** → TCP 9100, TCP 631 → **Pass**
4. **Force DNS via Pi-hole** (Option 2)  
   - Allow VLAN net → **<PUBLIC-IP-REDACTED>:53/853**  
   - **Block** VLAN net → **any:53/853** (place below allow)
5. **Internet Allow**  
   - VLAN net → **any** (after blocks/exceptions)

### 3) Xbox UPnP (VLAN 30 only)
- Services → **UPnP & NAT-PMP** → Interface: VLAN30 → Allow <PUBLIC-IP-REDACTED>/24

### 4) mDNS Across VLANs (for Home Assistant)
- Install **Avahi** (pfSense package) or run Avahi in a small LXC  
- Enable **reflector** between IOT/USERS ↔ SERVICES (or as desired)

---

## 🧪 Testing Checklist
- DHCP on each VLAN assigns correct gateway/DNS.
- From USERS, IOT, XBOX:  
  - 
slookup github.com → **server** shows Pi-hole or pfSense (per option).  
  - Internet works; direct <PUBLIC-IP-REDACTED>:53 is **blocked**.  
- Printer reachable from USERS/XBOX/IOT only on 9100/631.  
- IOT devices **cannot** ping USERS subnets (blocked RFC1918), but can reach printer if allowed.  
- Xbox NAT Type **Open** with UPnP on VLAN30 only.  
- Home Assistant discovers devices (mDNS reflected) or manual integrations work.

---

## 📷 Screenshots to Capture
- Pi-hole dashboard (no real client IPs), Top Clients masked  
- pfSense: DHCP DNS settings per VLAN  
- pfSense: Rules showing DNS block/allow order  
- Avahi/reflector config  
- HA Integrations page

---

## ✅ Status
VLANs enforced, DNS filtering via Pi-hole, HA online, least-privilege rules with targeted exceptions.




