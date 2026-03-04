## Day 6: NAT & Connection Tracking (Linux Router)

**Objective:** Configure a Linux machine to act as a router using Network Address Translation (NAT) and IP forwarding, and troubleshoot connection tracking for outbound internet access.

**Focus areas:**

- IP forwarding
- NAT (MASQUERADE)
- POSTROUTING chain
- Connection tracking (conntrack)
- Debugging NAT failures

## Lab Architecure
```text
| Machine | IP             | Role            |
| ------- | -------------- | --------------- |
| VM1     | 192.168.122.10 | Router (WAN)    |
| VM1     | 192.168.100.1  | LAN Gateway     |
| VM2     | 192.168.100.10 | Internal Client |
```
###Why NAT Is Required

Internal machines use private IP ranges:
```
192.168.x.x
```
The public internet does not route these addresses.
