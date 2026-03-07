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

## Step 1 – Enable IP Forwarding

Router must allow packet forwarding.

Check:
```bash
sysctl net.ipv4.ip_forward
```
Enable:
```
sysctl -w net.ipv4.ip_forward=1
```
## Step 2 – Enable NAT (Masquerading)

On VM1 run:
```
iptables -t nat -A POSTROUTING -o enp1s0 -j MASQUERADE
```
Explanation:
```text
| Option        | Meaning                              |
| ------------- | ------------------------------------ |
| `-t nat`      | Use NAT table                        |
| `POSTROUTING` | Modify packet after routing decision |
| `-o enp1s0`   | Outgoing WAN interface               |
| `MASQUERADE`  | Replace source IP with interface IP  |

```
## Step 3 – Test Internet Access

From VM2:
```
ping 8.8.8.8
curl google.com
```
Expected result:
- VM2 can access the internet through the router.

Observe NAT with tcpdump

On VM1:
```
tcpdump -i enp1s0 icmp
```
You should see packets leaving with:
```
SRC = 192.168.122.10
```
instead of the internal address.

This confirms NAT is rewriting the source IP.

### Failure Scenario – NAT Disabled

Flush NAT rules:
```
iptables -t nat -F
```
Now VM2 cannot reach the internet.

Reason:

Packets leave with private IP:
```
SRC = 192.168.100.10
```
External servers cannot route replies to private addresses.

