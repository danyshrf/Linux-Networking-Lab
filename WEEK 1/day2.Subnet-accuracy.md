# Day 2 – Static IP & Subnet Accuracy (Asymmetric Communication)

Understand how incorrect subnet masks and gateway configurations cause:
* One-way communication
* ARP inconsistencies
* Asymmetric routing behavior

**Focus:** How Linux decides whether a host is local or remote.

---

## Lab Architecture

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
```

Result: Ping between VM1 and VM2 still worked.

Why?
A /25 splits the subnet into two halves:
- 192.168.100.0 – 127
- 192.168.100.128 – 255

Both ```192.168.100.10``` and ```192.168.100.1``` fall into the first half. Both machines still believe they are in the same subnet, resulting in no communication failure.

## Incident  2 – True Subnet Mismatch

**Action:** Changed VM2 into the upper half of the /25 subnet.

```Bash
ip addr del 192.168.100.10/25 dev enp7s0
ip addr add 192.168.100.130/25 dev enp7s0
```
Current State:

- VM2 Subnet: 192.168.100.128 – 192.168.100.255
- Router (VM1): 192.168.100.1/24 (Sees 0-255 as local)
