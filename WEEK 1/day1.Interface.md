## DAY 1 - Interface State & IP Removal Failure 

### Scenario

Simulated network failure by:

- Bringing interface down
- Removing IP address
- Testing connectivity impact
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
