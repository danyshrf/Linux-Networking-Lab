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

## Netfilter Chain Model

| Chain   | Applies To                          |
| ------- | ----------------------------------- |
| INPUT   | Traffic destined to the router      |
| OUTPUT  | Traffic originating from the router |
| FORWARD | Traffic passing through the router  |

rule:
```bash
If router is destination → INPUT
If router is source → OUTPUT
If router is middleman → FORWARD
```

###Experiment 1 – Block Forwarding

On VM1:
```bash
iptables -P FORWARD DROP
```
Observations
### VM2 → VM1

Works
Reason: Packet hits INPUT chain.

### VM2 → VM3

Fails
Reason: Packet hits FORWARD chain → policy DROP.

### VM1 → VM2

Works
Reason: OUTPUT chain still ACCEPT.

## Experiment 2 – Block INPUT

On VM1:
```bash
iptables -P INPUT DROP
```

Observations
### VM2 → VM1

Fails
Reason: Packet hits INPUT → DROP.

### VM1 → VM2

Fails
Reason: Request leaves (OUTPUT OK), but reply hits INPUT → DROP.

### VM2 → VM3

Works
Reason: Transit traffic uses FORWARD, not INPUT.

