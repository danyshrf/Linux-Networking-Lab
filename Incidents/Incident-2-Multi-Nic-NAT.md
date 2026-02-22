## Linux Multi-NIC Router Lab – NAT & Routing Debugging
### Project Overview

I built a multi-network setup using:

Ubuntu 24.04 (host)

RHEL 9.7 (VM1 – Router)

RHEL 9.7 (VM2 – App Server)

RHEL 9.7 (VM3 – Client)

The goal was to design and debug a routed network where:
```
VM3 (Client) → VM1 (Router/NAT) → Ubuntu Host NAT → Internet
```
