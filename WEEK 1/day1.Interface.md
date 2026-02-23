## DAY 1 - Interface State & IP Removal Failure 

### Scenario

Simulated network failure by:

- Bringing interface down
- Removing IP address
- Testing connectivity 
- Observing ARP behavior under firewall block

### Failure Simulation 1 – Interface Down
```bash
ip link set enp1s0 down
```
Observed:

- ```UP``` flag removed
- ```LOWER_UP``` removed
- Ping failed immediately

Root Cause:
Interface administratively disabled → No Layer 1 transmission.

### Failure Simulation 2 – Remove IP
```bash
ip addr del 192.168.100.10/24 dev enp1s0
```
Observed:

- Interface still ```UP```
- ```LOWER_UP``` present
- No ```inet``` entry
- Ping failed with "Network unreachable"

Root Cause:
No source IP available → Layer 3 routing decision failed.

### Failure Simulation 3 – Firewall Blocking ICMP

On router:
```bash
firewall-cmd --add-icmp-block=echo-request
```
Observed:
- ARP resolved successfully
- ip neigh showed REACHABLE
- ICMP request seen in tcpdump
- No ICMP reply

Root Cause:
Firewall dropped ICMP at Layer 3.

### Packet Flow Analysis
```text
ARP (L2) → Success
ICMP (L3) → Dropped by firewall
```

## Key insight:
- ARP operates at Layer 2 and bypasses IP firewall filtering.
- Key Learnings
- UP = administrative state
- LOWER_UP = physical link detected
- Removing IP breaks routing, not Layer 2
- Firewall does not affect ARP
- Ping failure does not equal network failure
- 
### Command I Used: 
```text
ip link show
ip addr show
ip route
ip neigh
ethtool enp1s0
tcpdump -i enp1s0 icmp
firewall-cmd --list-all
```
