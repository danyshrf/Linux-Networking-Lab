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

Observed Behavior
VM1 → VM2 (Success)

- The ping request reaches VM2.
- Why: VM1 uses a ```/24``` mask and believes all ```192.168.100.x``` addresses are local. It ARPs for VM2 and successfully sends the packet.


VM2 → VM1 (Failure)

- The ping reply fails.
- Why: VM2 sees the router IP ```(192.168.100.1)``` as remote because it does not fall within its ```/25``` range (128-255). VM2 attempts to send the reply to its default gateway instead of directly to VM1.

## Incident 3 – Wrong Gateway
**Action:** Configured VM2 with a non-existent gateway.

```Bash
ip route add default via 192.168.100.200
```
(No device exists at .200)

Observed Behavior
Checking ARP on VM2:

```Bash
ip neigh
# Output shows: 192.168.100.200 INCOMPLETE
```
