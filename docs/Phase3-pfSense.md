---
title: Phase 3 – pfSense
nav_order: 30
---
# Phase 3 – pfSense VM Deployment

## 🎯 Objective
Deploy **pfSense** as a VM on Proxmox to provide routing, firewall, DHCP/DNS, and VLAN segmentation for FrankenServer.  
This phase sets up WAN/LAN, VLAN interfaces, DHCP scopes, baseline firewall rules, and service exceptions (printer access, Xbox UPnP).

---

## 🧩 Topology & Addressing (planned)
- **Switch:** Ruckus ICX7150-C12P
- **AP:** Ruckus R650 (multiple SSIDs)
- **pfSense VM:** 2 vNICs (VirtIO)
  - **WAN** → bridged to physical NIC connected to ISP modem
  - **LAN trunk** → bridged to physical NIC connected to the switch (802.1Q VLAN trunk)

### VLAN plan (you can keep these defaults)
| VLAN | Name         | Subnet           | Gateway (pfSense) | Notes                                        |
|-----:|--------------|------------------|-------------------|----------------------------------------------|
| 10   | AP-MGMT      | <IP-REDACTED>/24    | <IP-REDACTED>        | AP mgmt/native for AP uplinks if needed      |
| 20   | USERS        | <IP-REDACTED>/24    | <IP-REDACTED>        | PCs/printer access                           |
| 30   | XBOX-GAMING  | <IP-REDACTED>/24    | <IP-REDACTED>        | UPnP allowed; Internet only                  |
| 40   | IOT          | <IP-REDACTED>/24    | <IP-REDACTED>        | Block lateral; allow Internet                |
| 60   | SERVICES     | <IP-REDACTED>/24    | <IP-REDACTED>        | Shared services (e.g., **Printer**)          |
| 99   | MGMT         | <IP-REDACTED>/24    | <IP-REDACTED>        | Switch/AP/pfSense management (optional)      |

> Printer will live at **<IP-REDACTED>** in VLAN 60 so all VLANs can print via tight firewall rules.

---

## 🖥️ VM Specs (pfSense)
- 2 vCPU, 2–4 GB RAM, 20–32 GB disk (VirtIO SCSI)
- NIC model: **VirtIO (paravirtualized)**
- Boot from pfSense ISO; after install, remove ISO and enable Q35/UEFI if preferred.

---

## 🛠️ Proxmox Prep
1. **Upload ISO**: *Proxmox → local → ISO Images → Upload* (pfSense-CE-*.iso).
2. **Bridges**:
   - mbr0 → **WAN** bridge attached to the NIC that goes to your modem.
   - mbr1 → **LAN** bridge attached to the NIC that goes to the switch (802.1Q trunk).
   - Both bridges: set **VLAN Aware = Yes**.

> If you have only one physical NIC for now, you can trunk all VLANs on mbr0 and use VLAN tags on the pfSense LAN vNIC. The steps below assume two NICs (recommended).

---

## 📦 Create the VM
**Proxmox → Create VM**
- **OS**: Use pfSense ISO
- **System**: Q35/UEFI (or default SeaBIOS is fine), Machine type Q35
- **Disks**: 32 GB, SCSI, VirtIO SCSI controller
- **CPU/RAM**: 2 cores, 4096 MB
- **Network**:
  - **net0 (WAN)**: Bridge mbr0, VirtIO, *no VLAN tag*
  - **net1 (LAN trunk)**: Bridge mbr1, VirtIO, *no VLAN tag* (the trunk lands on this port)

Start VM and perform the pfSense install.

---

## 🔌 Initial Interface Assignment (pfSense console)
During the wizard:
- Assign **WAN** to the interface backed by 
et0 (VirtIO).
- Assign **LAN** to the interface backed by 
et1 (VirtIO).
- Set LAN temporary IP (e.g., <IP-REDACTED>/24) so you can reach the web GUI.

Browse to https://<IP-REDACTED> and run the setup wizard:
- Set hostname (e.g., pfsense), domain (williams.home.arpa), time/NTP.
- WAN: DHCP (typical for ISP).
- LAN: keep <IP-REDACTED>/24 for mgmt or change to your preferred LAN.

---

## 🧱 Create VLAN Interfaces (pfSense)
**Interfaces → Assignments → VLANs → Add**
- Parent interface: **LAN (net1)**  
- Create VLANs **10, 20, 30, 40, 60, 99**.

**Interfaces → Assignments**  
- Add each new VLAN as an OPT interface → click each → **Enable**, set:
  - Static IPv4:
    - VLAN10 → <IP-REDACTED>/24
    - VLAN20 → <IP-REDACTED>/24
    - VLAN30 → <IP-REDACTED>/24
    - VLAN40 → <IP-REDACTED>/24
    - VLAN60 → <IP-REDACTED>/24
    - (MGMT VLAN99 may stay on native LAN if you prefer; otherwise give it <IP-REDACTED>/24 on VLAN99)

---

## 🧮 DHCP Servers
**Services → DHCP Server**
- Enable DHCP on: VLAN10/20/30/40/60 (and 99 if desired).
- Example ranges:
  - VLAN10: <IP-REDACTED>–<IP-REDACTED>
  - VLAN20: <IP-REDACTED>–<IP-REDACTED>
  - VLAN30: <IP-REDACTED>–<IP-REDACTED>
  - VLAN40: <IP-REDACTED>–<IP-REDACTED>
  - VLAN60: <IP-REDACTED>–<IP-REDACTED>
- **Static mapping** (Printer): VLAN60 → <IP-REDACTED> to the printer MAC.

> DNS: keep pfSense (Unbound) for now; in Phase 4 you can forward to Pi-hole.

---

## 🔐 Baseline Firewall Rules (per interface)
**Firewall → Aliases**
- **NETS_ALLOWED_TO_PRINTER** = <IP-REDACTED>/24, <IP-REDACTED>/24, <IP-REDACTED>/24, <IP-REDACTED>/24
- **PRINTER_IP** = <IP-REDACTED>
- **PRINTER_PORTS** = TCP 9100, TCP 631

**Firewall → Rules → (each VLAN)**
1. **Allow Essential** (top):
   - Proto **UDP**: VLAN net → pfSense **53, 67-68** (DNS/DHCP)
   - Proto **TCP**: VLAN net → pfSense **53, 80, 443** (web admin/DNS if needed)
   - Proto **UDP**: VLAN net → pfSense **123** (NTP)
2. **Allow to Internet**
   - Source: VLAN net → Destination: **any** → *Block laterally first if you prefer stricter stance.*
3. **Block Inter-VLAN (default deny)**
   - Source: VLAN net → Destination: **RFC1918** → **Block**
   - Place **above** the Internet rule if you want isolation; then add specific allows below.
4. **Allow Printing (exception)**
   - Source: **NETS_ALLOWED_TO_PRINTER** → Destination: **PRINTER_IP** → **PRINTER_PORTS** → **Pass**
5. **IOT tighter** (VLAN40):
   - Only Essential + Internet; **no** access to other RFC1918 except printer rule if desired.

**Xbox UPnP (VLAN30 only)**
- **Services → UPnP & NAT-PMP**: Enable on **VLAN30** interface only.
- Set ACL to allow <IP-REDACTED>/24 and deny others.

---

## 🔄 NAT
- **Firewall → NAT → Outbound**: keep **Automatic** unless you need manual rules.
- **Reflection**: typically **disabled** unless you need hairpin NAT.

---

## 🌐 Switch & AP Notes (quick)
- Trunk from pfSense LAN bridge → switch port: allow VLANs **10,20,30,40,60,99**.
- Access ports:
  - Printer port (e.g., port 6) → **VLAN60 (access)**.
  - Xbox SSID → map to **VLAN30**.
  - IoT SSID → map to **VLAN40**.
  - User SSID / wired PCs → **VLAN20**.
  - AP mgmt (if used) → **VLAN10** or MGMT VLAN99.

---

## 🧪 Testing
- Client on each VLAN gets correct DHCP scope/gateway.
- From a user VLAN, print to <IP-REDACTED>.
- Xbox NAT Type → **Open** (with UPnP on VLAN30).
- Inter-VLAN isolation verified (cannot ping other VLAN gateways).
- Internet works from each VLAN.

---

## 🧰 Backup & Repo
- **Diagnostics → Backup & Restore → Download config.xml**
- **Do not commit secrets**. If you want a sanitized copy in Git, redact passwords/keys before saving to configs\pfsense-backup.xml.

---

## 📷 Screenshots to capture
- Interface Assignments (including VLANs)
- DHCP Server pages per VLAN
- Firewall → Aliases
- Firewall → Rules (one screenshot per VLAN)
- UPnP interface selection
- Dashboard summary

---

## ✅ Status
pfSense VM deployed with WAN/LAN, VLANs, DHCP scopes, core firewall rules, printer exception, and (optional) Xbox UPnP.  
Next: Phase 4 – VLAN testing, Pi-hole integration, and Home Assistant.






