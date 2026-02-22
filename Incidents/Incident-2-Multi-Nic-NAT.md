## Linux Multi-NIC Router Lab – NAT & Routing Debugging
### Project Overview

### I built a multi-network setup using:

* Ubuntu 24.04 (host)

* RHEL 9.7 (VM1 – Router)

* RHEL 9.7 (VM2 – App Server)

* RHEL 9.7 (VM3 – Client)

The goal was to design and debug a routed network where:
```
VM3 (Client) → VM1 (Router/NAT) → Ubuntu Host NAT → Internet
```
### Topology
```text
                    Internet
                        |
                Ubuntu Host (libvirt NAT)
                        |
                192.168.122.0/24
                        |
                 VM1 (Router)
              ┌────────┴────────┐
      192.168.100.0/24    192.168.200.0/24
           VM2 (Server)         VM3 (Client)
```

