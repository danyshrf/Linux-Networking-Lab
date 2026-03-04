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
## Step 3 – Test Internet Access

From VM2:
```
ping 8.8.8.8
curl google.com
```
Expected:

- ICMP replies received 
- HTTP requests succeed

### Connection Tracking

Linux maintains a state table for NAT translations.

View connection tracking entries:
```
conntrack -L
```
or
```
cat /proc/net/nf_conntrack
```
Example entry:
```
tcp ESTABLISHED
src=192.168.100.10 dst=8.8.8.8
```
This table allows return packets to be mapped back to the original internal host.

## Failure Scenario 1 – NAT Rule Removed

Remove NAT rules:
```
iptables -t nat -F
```
Now VM2 cannot reach internet.

Reason:

Packets leave router with private source IP:
```
SRC: 192.168.100.10
```
External hosts cannot route replies to private addresses.

## Failure Scenario 2 – IP Forwarding Disabled

Disable forwarding:
```
sysctl -w net.ipv4.ip_forward=0
```
Result:

- Router refuses to forward packets between interfaces.
- Traffic from VM2 never leaves the router.

### Debugging NAT Issues

When internal hosts cannot reach internet:

**Check gateway**
```
ip route
```
**Verify IP forwarding**
```
sysctl net.ipv4.ip_forward
```
**Inspect NAT rules**
```
iptables -t nat -L -n -v
```
**Confirm outgoing interface**
```
ip a
```
Observe packets with tcpdump
```bash
tcpdump -i enp1s0 icmp
```
**ey observation:**
```bash
| Packet Source  | Meaning         |
| -------------- | --------------- |
| 192.168.100.10 | NAT not applied |
| 192.168.122.10 | NAT working     |

```
## DNS vs Connectivity

During testing:
```bash
ping 8.8.8.8
```
worked, but:
```bash
curl google.com
```
failed.

Root cause: **DNS resolution failure.**

Connectivity to IP addresses was working, but domain names could not be resolved.

### Check DNS configuration:
```bash
cat /etc/resolv.conf
```
Fix:
```bash
echo "nameserver 8.8.8.8" > /etc/resolv.conf
```

## Key Mental Model

Internet access requires four components working together:
```bash 
LAN routing
↓
IP forwarding
↓
NAT translation
↓
Connection tracking
```
Breaking any of these prevents internet connectivity.
