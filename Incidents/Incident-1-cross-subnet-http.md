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
## How I Investigated

Step 1 – Verify Service
On VM2, confirm the web server is actually running and listening for traffic:

```bash
ss -tulnp | grep :80
Confirmed nginx is listening on 0.0.0.0:80.
```
Step 2 – Verify Routing Logic
On VM1 (Router), test how the kernel is routing the traffic:

```bash
ip route get 192.168.100.10 from 192.168.200.10
# Output: Network is unreachable
This indicated a Forwarding Information Base (FIB) failure.
```
Step 3 – Identify Kernel Source Validation
Checked reverse path filtering, a security feature that can sometimes drop legitimately routed packets:

```bash
sysctl net.ipv4.conf.enp7s0.rp_filter
```
```rp_filter``` was enabled. Disabled it temporarily:

```bash
sysctl -w net.ipv4.conf.all.rp_filter=0
Forwarding lookup succeeded after disabling.
```
Forwarding lookup succeeded after disabling.

Step 4 – Packet-Level Verification
On VM1 (Router), run packet captures on both internal interfaces to watch the traffic flow:

```Bash
tcpdump -i enp8s0 tcp port 80
tcpdump -i enp7s0 tcp port 80
```
Observed:

SYN packet entered the router.

SYN packet forwarded to LAN1.

No SYN-ACK packet returned.

Step 5 – Check Destination Host Firewall
On VM2 (App Server), check if the local firewall is dropping the incoming request:

```Bash
systemctl status firewalld
```
```firewalld``` was active. Stopped it temporarily:

```bash
systemctl stop firewalld
```
HTTP traffic immediately worked.

## Root Causes
Reverse path filtering (```rp_filter```) on the router was blocking the forwarded traffic.

```firewalld``` on VM2 was actively blocking web traffic coming from the ```192.168.200.0/24``` subnet.

## Resolution
Disabled rp_filter for the multi-interface router to allow proper packet forwarding.

Added a permanent ```firewalld``` rule on the App Server to permit the traffic, rather than leaving the firewall disabled:

```Bash
firewall-cmd --permanent --add-source=192.168.200.0/24
firewall-cmd --permanent --add-service=http
firewall-cmd --reload
```
