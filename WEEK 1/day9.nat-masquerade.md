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

# 🧠 Why NAT Is Needed

Internal machines use **private IP addresses**:
