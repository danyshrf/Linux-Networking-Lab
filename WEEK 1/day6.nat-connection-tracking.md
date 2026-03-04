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
### Why NAT Is Required

Internal machines use private IP ranges:
```
192.168.x.x
```
The public internet does not route these addresses.

When VM2 sends traffic:
```code
SRC: 192.168.100.10
DST: 8.8.8.8
```
The router rewrites it to:
```
SRC: 192.168.122.10
DST: 8.8.8.8
```
This allows the internet to send replies back to the router.
The router then maps the reply back to the internal machine.

## Step 1 – Enable IP Forwarding

Linux does not behave as a router unless forwarding is enabled.

Check:
```bash
sysctl net.ipv4.ip_forward
```
Enable:
```bash
sysctl -w net.ipv4.ip_forward=1
```
Permanent configuration:
```bash
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
```
## Step 2 – Configure NAT

Add NAT rule on router:
```bash
iptables -t nat -A POSTROUTING -o enp1s0 -j MASQUERADE
```
```text
| Option        | Meaning                                      |
| ------------- | -------------------------------------------- |
| `-t nat`      | Use NAT table                                |
| `POSTROUTING` | Modify packet after routing decision         |
| `-o enp1s0`   | Outgoing WAN interface                       |
| `MASQUERADE`  | Replace source IP with router's interface IP |
```

