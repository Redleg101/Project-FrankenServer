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

Phase 3 · Step 1 — pfSense ISO Preparation

System: Proxmox Host (Dell OptiPlex XE2)
Actions Completed:
1.	Navigated to pfSense official website → downloaded latest pfSense Community Edition ISO (amd64).
2.	Verified ISO checksum against published SHA256 to ensure file integrity.
3.	Accessed Proxmox Web GUI → selected local storage (ISO Images).
4.	Uploaded pfSense ISO (pfSense-CE-2.7.2-RELEASE-amd64.iso) to Proxmox.
5.	Confirmed upload success and ISO visible in Proxmox storage list.
Configuration Adjustments:
•	Verified Proxmox storage (local) has sufficient space for multiple VM ISOs.
•	Confirmed network bridges exist for future pfSense assignment:
o	vmbr0 (management/WAN candidate).
o	vmbr1 (planned LAN bridge).
Result:
•	pfSense ISO successfully downloaded and uploaded to Proxmox.
•	System now ready to proceed to VM creation (Step 2) once new NIC is installed.
________________________________________

Phase 3 · Step 2 — pfSense VM Creation (Paused)

System: Proxmox Host (Dell OptiPlex XE2)
Actions Attempted:
1.	Began VM creation in Proxmox:
o	Name: pfSense-VM01.
o	Resources: 2 vCPUs, 4GB RAM, 20GB disk (VirtIO).
o	ISO Boot Media: pfSense CE 2.7.2.
2.	Attempted to assign two NICs:
o	WAN → vmbr0.
o	LAN → vmbr1.
3.	Verified PCI pass-through options available for physical NIC assignment.
Issue Encountered:
•	Installed LinksTek X540-T2 NIC failed initialization.
•	ixgbe driver errors: EEPROM checksum failures, Adapter removed / error -5.
•	Interfaces not visible in Proxmox → pfSense VM cannot be fully configured.
Result:
•	Step paused. VM skeleton exists in Proxmox but network assignment incomplete.
•	Awaiting replacement Intel OEM NIC (X540-T2 or X550-T2).

Phase 3 – Interface Assignment Plan
Available NICs
•	Intel I217-LM (00:19.0) → Onboard 1GbE (built into motherboard)
•	Broadcom NetXtreme BCM5722 (01:00.0) → PCIe add-in 1GbE card
•	Intel X540-AT2 (05:00.0 / 05:00.1) → Dual 10GbE PCIe (faulty; returning)
________________________________________
Recommended Role Assignments
1.	Intel I217-LM (Onboard) → Proxmox Management
o	Dedicated to host management, so you can always reach Proxmox even if pfSense breaks.
o	Assigned to vmbr0 in Proxmox.
2.	Broadcom BCM5722 → pfSense WAN
o	Connect this port directly to your ISP router/modem (or upstream router).
o	Passed through to the pfSense VM as the WAN interface.
3.	Replacement NIC (Single-Port Intel or Dual-Port Intel) → pfSense LAN
o	Connect this port to your Ruckus ICX7150-C12P switch.
o	Passed through to the pfSense VM as the LAN interface.
o	This will provide connectivity for all VLANs, lab gear, and home devices.
________________________________________
Benefits of This Layout
•	Separation of duties → Proxmox mgmt never depends on pfSense.
•	pfSense has dedicated WAN + LAN → required for proper firewall/routing.
•	Scalable → later you can upgrade the LAN NIC to 10GbE and connect it to the ICX SFP+ uplink.
________________________________________
👉 Once you install the new NIC, we’ll update your Proxmox Bridges and map them:
•	vmbr0 → I217-LM (mgmt)
•	vmbr1 → WAN (Broadcom)
•	vmbr2 → LAN (new Intel NIC)

Phase 3 · Step 2 — pfSense VM Creation (NIC Install & Prep)

System: Dell OptiPlex XE2 (Proxmox Host)
Actions Completed:
1.	Installed TP-Link TX201 2.5GbE PCIe NIC (Realtek RTL8125 chipset) into available PCIe slot.
2.	Booted system and verified detection with:
3.	lspci -nn | grep -i ethernet
Output confirmed presence of:
o	Intel I217-LM (onboard, 1GbE)
o	Broadcom BCM5722 (PCIe, 1GbE)
o	Realtek RTL8125 (PCIe, 2.5GbE)
4.	Confirmed all NICs visible to Proxmox kernel with correct driver binding.
Configuration Adjustments:
•	Planned role assignment:
o	Intel I217-LM → Proxmox Management (vmbr0)
o	Broadcom BCM5722 → pfSense WAN (vmbr1)
o	Realtek RTL8125 → pfSense LAN (vmbr2)
•	Verified available PCIe slot bandwidth (x4 electrical) adequate for 2.5GbE NIC.
Result:
•	New NIC successfully installed and recognized.
•	System ready to proceed with pfSense VM build and network bridge assignment.
________________________________________
👉 Next step will be to create:
1.	vmbr1 for Broadcom (WAN)
2.	vmbr2 for Realtek (LAN)
and then attach both to the pfSense VM alongside the ISO we uploaded earlier.

Phase 3 · Step 3 — Proxmox Bridge Setup for pfSense

System: Dell OptiPlex XE2 (Proxmox Host)
Actions Completed:
1.	Logged into Proxmox Web GUI → Node → System → Network.
2.	Verified existing management bridge:
o	vmbr0 bound to Intel I217-LM (onboard NIC) → 192.168.1.50/24.
3.	Edited vmbr1 to bind Broadcom BCM5722 (enp1s0) as the pfSense WAN bridge.
4.	Created new vmbr2 and bound Realtek RTL8125 (enp5s0) as the pfSense LAN bridge.
5.	Left IP/CIDR/Gateway fields blank for both vmbr1 and vmbr2 (pfSense will provide addressing).
6.	Applied configuration through Proxmox interface.
Configuration Adjustments:
•	Bridge Assignments now aligned with project plan:
o	vmbr0 → Intel I217-LM (Proxmox mgmt)
o	vmbr1 → Broadcom BCM5722 (pfSense WAN)
o	vmbr2 → Realtek RTL8125 (pfSense LAN)
•	Verified all bridges show as Active and bound to correct physical devices.
Notes / Warnings:
•	Proxmox displayed a warning:
•	WARN: missing 'source /etc/network/interfaces.d/sdn' directive for SDN support!
o	This relates only to Proxmox SDN (Software Defined Networking).
o	SDN not in use for this project → safe to ignore.
o	Optionally silenced by adding:
o	source /etc/network/interfaces.d/sdn
to /etc/network/interfaces.
Result:
•	Proxmox networking successfully configured with 3 bridges.
•	System ready to proceed with pfSense VM creation and NIC assignment in the next step.
________________________________________
👉 Next up is Phase 3 · Step 4 — pfSense VM configuration (DHCP, firewall rules, NAT).

Phase 3 · Step 4 — pfSense Installation & Initial Configuration

System: Dell OptiPlex XE2 (Proxmox Host)
Actions Completed:
1.	Booted pfSense VM from ISO.
2.	Completed pfSense installer:
o	Chose default installation settings.
o	Partitioned 32GB SATA virtual disk.
o	Installed pfSense 2.7.2-RELEASE (amd64).
o	Rebooted VM into pfSense.
3.	Verified NIC assignments:
o	WAN (em0) → Broadcom BCM5722 (vmbr1).
o	LAN (em1) → Realtek RTL8125 (vmbr2).
4.	pfSense LAN automatically assigned default IP → 192.168.10.1/24.
Configuration Adjustments:
•	Console displayed pfSense main menu.
•	Confirmed webConfigurator available on LAN at:
https://192.168.10.1
•	Login credentials:
o	Username: admin
o	Password: pfsense
Result:
•	pfSense VM installed and operational inside Proxmox.
•	LAN access functional with default addressing.
•	WAN interface detected but not yet configured.
________________________________________
👉 Next step will be to:
•	Access pfSense web GUI via a client connected to LAN (or via virtual test VM in Proxmox bridged to vmbr2).
•	Run the Initial Setup Wizard:
o	Change admin password.
o	Configure WAN interface (static/DHCP).
o	Confirm outbound NAT.
o	Enable DHCP on LAN.

Phase 3 · Step 5 — pfSense LAN IP & DHCP Configuration

System: Dell OptiPlex XE2 (Proxmox Host)
Actions Completed:
1.	Accessed pfSense console menu.
2.	Selected Option 2 → Set interface(s) IP address.
3.	Configured LAN (em1) with static IP:
o	IPv4: 192.168.10.1
o	Subnet: /24 (255.255.255.0)
o	Upstream gateway: none
o	IPv6: disabled
4.	Enabled DHCP server on LAN:
o	Range: 192.168.10.100 – 192.168.10.200
5.	Left webConfigurator as HTTPS only.
Configuration Adjustments:
•	WAN (em0): still unconfigured, will be set via DHCP or static during wizard.
•	LAN (em1): now reachable at 192.168.10.1/24.
•	pfSense Web GUI available at:
👉 https://192.168.10.1
Result:
•	pfSense LAN interface properly configured.
•	DHCP active, ready to assign addresses to clients.
•	System ready to run pfSense Setup Wizard via webConfigurator.
________________________________________
👉 Next Step: Open a browser on a VM or device connected to LAN (vmbr2) → log in to https://192.168.10.1 → complete Setup Wizard (hostname, DNS, WAN type, LAN subnet, admin password).

Phase 3 · Step 6 — pfSense WebConfigurator Access & Setup Wizard

System: Dell OptiPlex XE2 (Proxmox Host)
Actions Completed:
1.	Reassigned pfSense interfaces via console (Option 1):
o	WAN → em0 (Broadcom, vmbr1).
o	LAN → em1 (Realtek, vmbr2).
o	Removed any optional interfaces.
o	Confirmed new assignments applied successfully.
2.	Connected laptop to LAN (Realtek NIC → TP-Link switch).
o	Laptop received DHCP lease (192.168.10.102/24, gateway 192.168.10.1).
3.	Accessed pfSense WebConfigurator for the first time:
o	URL: http://192.168.10.1 (initially bound to HTTP).
o	Login successful using default credentials:
	Username: admin
	Password: pfsense.
4.	Ran the Setup Wizard:
o	General Setup: Hostname = pfsense, Domain = frankenserver.local.
o	DNS Servers: 1.1.1.1 and 8.8.8.8 (unchecked “Override DNS”).
o	Time Zone: America/New_York, NTP = default.
o	WAN: Configured as DHCP → Spectrum router provided IP.
o	LAN: Confirmed static IP = 192.168.10.1/24; DHCP range 192.168.10.100–192.168.10.200.
o	Admin Password: Changed from default to secure password.
5.	Completed wizard and rebooted pfSense VM.
Configuration Adjustments:
•	pfSense LAN now handing out DHCP correctly.
•	WAN confirmed live on Spectrum router via DHCP.
•	WebConfigurator bound and accessible.
Result:
•	pfSense fully accessible from LAN side via browser.
•	Setup Wizard complete → ready for firewall rule customization, NAT testing, and VLAN buildout in Phase 4.
•	Internet routing through pfSense ready for testing with client devices.

Phase 3 · Step 7 — PC DNS/Network Stack Issue & Fix

System: Dell OptiPlex XE2 (Proxmox Host) + Windows 10 PC
Issue Observed:
•	After pfSense setup, main PC could not access the internet.
•	Symptoms:
o	Showed “Connected, no internet” both on Spectrum router and pfSense LAN.
o	Could ping external IPs (8.8.8.8, 1.1.1.1) ✅.
o	Could not resolve DNS names (e.g., nslookup google.com timed out ❌).
o	Laptop on same network worked fine.
Actions Completed:
1.	Verified IP addressing:
o	Spectrum router → PC received 192.168.1.42 / 192.168.1.1 gateway.
o	pfSense LAN → Laptop received 192.168.10.102 / 192.168.10.1 gateway.
o	Confirmed routing functional but DNS queries failed.
2.	Attempted basic DNS fixes:
o	Flushed and renewed DHCP lease.
o	Manually set DNS servers (8.8.8.8, 1.1.1.1).
o	Flushed DNS cache (ipconfig /flushdns).
o	❌ Problem persisted.
3.	Performed full TCP/IP & Winsock reset:
4.	netsh int ip reset
5.	netsh winsock reset
o	Rebooted PC.
6.	Post-reboot validation:
o	nslookup google.com succeeded ✅.
o	ping google.com resolved and replied ✅.
o	Websites including ChatGPT loaded successfully ✅.
Result:
•	Network stack corruption on PC fully resolved.
•	DNS resolution and web access restored.
•	PC and laptop now both operational across Spectrum and pfSense networks.

Phase 3 · Step 8 — Verify Internet Routing Through pfSense

System: Dell OptiPlex XE2 (Proxmox Host)
Actions Completed:
1.	Moved main PC to pfSense LAN (via TP-Link switch uplinked to Realtek NIC).
2.	Verified IP configuration on PC:
o	IPv4 Address: 192.168.10.101
o	Subnet Mask: 255.255.255.0
o	Gateway: 192.168.10.1
o	DNS Server: 192.168.10.1
3.	Performed connectivity tests:
o	✅ Ping to pfSense LAN gateway (192.168.10.1) successful.
o	✅ Ping to external IP (8.8.8.8) successful.
o	✅ Ping to domain name (google.com) resolved and successful.
o	✅ Browser test to http://neverssl.com loaded successfully.
4.	Confirmed NAT and DNS resolution working end-to-end through pfSense.
Configuration Adjustments:
•	No further changes required. Default pfSense firewall and NAT rules allowed outbound traffic as expected.
Result:
•	Internet access fully functional for LAN clients via pfSense.
•	pfSense confirmed operational as primary routing device between LAN and Spectrum WAN.

