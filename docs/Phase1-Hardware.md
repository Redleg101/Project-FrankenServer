---
title: Phase 1 – Hardware
nav_order: 10
---
# Phase 1 – Hardware Build



\## 📝 Phase 1 – Notes \& Observations



\- \*\*Chassis:\*\* Dell OptiPlex XE2 (repurposed)  

\- \*\*Switch:\*\* Ruckus ICX7150-C12P  

\- \*\*NICs detected:\*\* Onboard Broadcom (`tg3`), single-port Realtek ✅  

\- \*\*NIC issue:\*\* LinksTek X540-T2 dual 10GbE showed \*EEPROM checksum invalid\* → decision: return/replace  

\- \*\*Drives installed:\*\* SSD (OS) + 3× HDD  

&nbsp; - \*\*SATA map (physical):\*\* HDD P5, HDD P6, ODD P4, ODD P5  

&nbsp; - \*\*Finding:\*\* Proxmox initially saw only \*\*2/4\*\* drives  

&nbsp; - \*\*Suspects:\*\* cabling/port mapping, BIOS SATA mode  

\- \*\*BIOS:\*\* Boot order verified; next step to confirm all disks visible in BIOS  

\- \*\*Power:\*\* Stable under load; fans/temps nominal



\### Troubleshooting log (highlights)

\- `dmesg | grep -i tg3` → onboard NIC up  

\- `lspci -nn | grep -i ethernet` → Realtek present, X540 enumerates but EEPROM fault  

\- Proxmox GUI → only 2 HDDs listed under Disks



\### Next actions

\- Re-seat/re-map SATA: test ports P4/P5/P6 one by one  

\- Ensure \*\*AHCI\*\* (not RAID/IDE) in BIOS for all SATA ports  

\- Replace/return the X540-T2; proceed with 1GbE NICs for Phase 2–3  

\- After drives fixed: create ZFS pool for VM storage



> \*\*Note:\*\* IPs/hostnames in docs are sanitized for security; production uses different ranges and credentials.




