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
### Incident Summary
VM3 could ping the router (192.168.200.1),
but could NOT reach the internet (8.8.8.8).

Error observed:

```
Destination Port Unreachable
```
At first glance:

* Routing looked correct
* NAT rule existed
* ip_forward was enabled

But internet traffic still failed.

### Root Causes Identified

This incident had multiple layered issues:

1️Duplicate Gateway IP (Critical)

The Ubuntu host bridge and VM1 were both assigned:
```
192.168.200.1
```
This caused:

* ARP instability
* Traffic being sent to the host instead of VM1

Fix:
Removed ```<ip>``` section from libvirt internal network XML.
Made it a pure L2 bridge:
```
<forward mode='none'/>
```
