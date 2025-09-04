---
title: Phase 2 – Proxmox
nav_order: 20
---
# Phase 2 – Proxmox Installation



Phase 2: Proxmox Install \& Configuration

Goal: Get Proxmox installed, stable, and ready for services.

•	Step 1: Update BIOS/Firmware \& enable virtualization (Done)

•	Step 2: Install Proxmox VE (Done)

•	Step 3: Configure Storage (check local + local-lvm, add extra drives if needed)

•	Step 4: Networking Setup (static IP confirmed, test connectivity from main PC)

•	Step 5: Enable Remote Management (confirm Web GUI access from LAN, optional Let’s Encrypt cert)

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_



**Phase 2 · Step 1 — BIOS \& Firmware Update**



System: Dell OptiPlex XE2

Actions Completed:

1\.	Prepared a FAT32-formatted USB flash drive.

2\.	Downloaded latest BIOS update from Dell Support (XE2A25.exe).

3\.	Copied BIOS update file to USB root directory.

4\.	Booted system using F12 → BIOS Flash Update utility.

5\.	Applied BIOS update successfully.

6\.	Verified update by entering BIOS Setup (F2) after reboot.

7\.	Diagnostics Run

Configuration Adjustments:

•	Boot Mode: Set to UEFI

•	Secure Boot: Disabled (for now; will revisit after hypervisor install)

•	Virtualization Support: Enabled (Intel VT-x, VT-d)

•	AC Recovery set to Power On.

•	Wake-on-LAN enabled.

•	Disabled unused devices (audio, serial).

Result:

•	BIOS successfully updated to latest revision.

•	System settings configured for virtualization and ready for next step.

•	Diagnostics Run (Optional Check)

o	Performed Dell ePSA hardware diagnostics.

o	All fans passed functional test (RPMs within range).

o	Thermal sensors normal:

	Hard drives: ~33–34 °C

	CPU Thermistor: 45 °C

	Ambient Thermistor: 23 °C

o	No hardware faults detected.

Result: Hardware confirmed stable for hypervisor deployment.

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_



**Phase 2 · Step 2 — Hypervisor Installation (Proxmox VE)**



System: Dell OptiPlex XE2

Actions Completed:

1\.	Created Proxmox VE bootable USB using Rufus in DD Image mode.

2\.	Booted OptiPlex XE2 via F12 Boot Menu, selected Install Proxmox VE (Graphical).

3\.	Installed Proxmox to primary hard drive with following settings:

o	Hostname (FQDN): williams.home.arpa

o	Management Interface: Intel NIC (eno1)

o	IP Address: <IP-REDACTED>/24

o	Gateway: <IP-REDACTED>

o	DNS Server: <IP-REDACTED>

4\.	Rebooted into Proxmox console.

5\.	Adjusted network cabling to Intel onboard NIC (eno1) to bring vmbr0 bridge UP.

6\.	Verified IP assignment and network link status with ip a.

7\.	Successfully accessed Proxmox Web GUI at:

o	https://<IP-REDACTED>:8006

8\.	Logged in with root account and confirmed system operational.

Notes:

•	Browser displayed a self-signed SSL warning on first login → bypassed successfully.

•	Proxmox displayed “No valid subscription” message → dismissed, system functions normally.

•	System now ready for VM and container configuration in Phase 3.

Result:

Proxmox VE installed and accessible via web interface. Networking confirmed operational through Intel NIC (eno1). System is ready for virtual machine deployment.

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_



**Phase 2 · Step 3 — Configure Storage**

•	Verified Proxmox boot SSD (/dev/sda) in use by system.

•	Wiped partitions from 3 × 1TB HDDs (/dev/sdb, /dev/sdc, /dev/sdd).

•	Initialized each HDD with GPT.

•	Added them into Proxmox as storage pools (hdd1, hdd2, hdd3).

•	Confirmed ~3 TB additional storage available for VMs/containers.



**Phase 2 · Step 4 — Networking Setup**

•	Verified vmbr0 bridge bound to Intel NIC (eno1) with static IP <IP-REDACTED>/24.

•	Successfully pinged router (<IP-REDACTED>), external DNS (<IP-REDACTED>), and internet host (google.com).

•	Confirmed Proxmox Web GUI accessible at https://<IP-REDACTED>:8006 from main PC.



**Phase 2 · Step 5 — Enable Remote Management**

•	Proxmox Web GUI tested and reachable from multiple devices on LAN.

•	Bookmarked management URL for quick access.

•	SSH tested from Windows PowerShell (ssh root@<IP-REDACTED>) → login with root password successful.

•	Confirmed FrankenServer now manageable via both Web GUI and SSH.

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_









