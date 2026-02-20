# Incident 01: Cross-Subnet HTTP Failure

> **TL;DR:** A client machine (VM3) could ping an app server (VM2) on a different subnet but couldn't access its web page. The investigation revealed two issues: 
> 1. The router's kernel was dropping forwarded packets due to a security setting (`rp_filter`).
> 2. The app server's local firewall was blocking HTTP traffic from outside its own network.

## Environment
This repository documents hands-on Linux networking failure simulations performed in a KVM lab. Focus areas include routing, firewalls, NAT, reverse path filtering, and packet-level debugging.

* **Hypervisor:** KVM on Ubuntu 24.04
* **VM1 (Router):** RHEL 9.7 
  * WAN: `192.168.122.10`
  * LAN1: `192.168.100.1`
  * LAN2: `192.168.200.1`
* **VM2 (App Server):** `192.168.100.10`
* **VM3 (Client):** `192.168.200.10`

### Topology
```text
VM3 (192.168.200.10)
        |
        |
VM1 Router
  LAN2: 192.168.200.1
  LAN1: 192.168.100.1
        |
        |
VM2 (192.168.100.10)
```
## Problem Statement
VM3 could ping VM2, but could not access the HTTP service on VM2.

Error observed:
```bash
curl 192.168.100.10
# Output: No route to host / Connection refused
```

