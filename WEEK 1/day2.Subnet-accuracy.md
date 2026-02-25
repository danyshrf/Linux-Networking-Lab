# Day 2 – Static IP & Subnet Accuracy (Asymmetric Communication)

Understand how incorrect subnet masks and gateway configurations cause:
* One-way communication
* ARP inconsistencies
* Asymmetric routing behavior

**Focus:** How Linux decides whether a host is local or remote.

---

## 🏗 Lab Architecture

| Machine | IP Address | Role |
| :--- | :--- | :--- |
| **VM1** | `192.168.100.1/24` | Router |
| **VM2** | `192.168.100.10` (Modified) | App Server |
| **VM3** | `192.168.200.10` | Client |

---

## Incident 1 – Subnet Mask Change (Working Case)

**Action:** Changed VM2 IP configuration.
```bash
ip addr del 192.168.100.10/24 dev enp7s0
ip addr add 192.168.100.10/25 dev enp7s0
