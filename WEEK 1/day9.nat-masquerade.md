# Day 9 – NAT (Masquerading)

## Goal

Understand how Linux routers perform **Network Address Translation (NAT)** using MASQUERADE.

Focus areas:

- NAT basics
- POSTROUTING chain
- Source IP rewriting
- Internal network internet access
- Observing NAT behavior

---


#  Lab Architecture

| Machine | IP | Role |
|--------|-------------|------|
| VM1 | 192.168.122.10 | Router (WAN) |
| VM1 | 192.168.100.1 | LAN Gateway |
| VM2 | 192.168.100.10 | Internal Client |

Goal:

VM2 should access the internet **through VM1**.

---

# Why NAT Is Needed

Internal machines use **private IP addresses**:

192.168.x.x
These addresses are **not routable on the internet**.

Example packet from VM2:
```
SRC: 192.168.100.10
DST: 8.8.8.8
```
If sent directly, the internet cannot send replies back.

The router must rewrite the source IP.

After NAT:
```
SRC: 192.168.122.10
DST: 8.8.8.8
```
Now the internet replies to the router, which forwards the response back to VM2.

Step 1 – Enable IP Forwarding

Router must allow packet forwarding.

Check:
```bash
sysctl net.ipv4.ip_forward
```
Enable:
```
sysctl -w net.ipv4.ip_forward=1
```

