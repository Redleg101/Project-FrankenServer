# Project FrankenServer





> **Note:** Addresses below are examples for documentation. Production uses different ranges and hostnames.

| VLAN | Name | Subnet | Gateway | Key Hosts |
|-----:|------|--------|---------|-----------|
| 20 | USERS | 10.100.20.0/24 | 10.100.20.1 | — |
| 30 | XBOX | 10.100.30.0/24 | 10.100.30.1 | — |
| 40 | IOT | 10.100.40.0/24 | 10.100.40.1 | — |
| 60 | SERVICES | 10.100.60.0/24 | 10.100.60.1 | Printer 10.100.60.10, Pi-hole 10.100.60.20 |
| 99 | MGMT | 10.100.99.0/24 | 10.100.99.1 | Switch/AP/pfSense |
