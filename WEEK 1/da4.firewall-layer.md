# Firewall Layer Deep Dive (iptables & Packet Flow Control)

Focus areas:

- INPUT vs OUTPUT vs FORWARD chains
- Default policies
- Transit vs local traffic
- How firewall differs from routing
- Packet path reasoning

| Machine | IP                            | Role       |
| ------- | ----------------------------- | ---------- |
| VM1     | 192.168.100.1 / 192.168.200.1 | Router     |
| VM2     | 192.168.100.10                | App Server |
| VM3     | 192.168.200.10                | Client     |
