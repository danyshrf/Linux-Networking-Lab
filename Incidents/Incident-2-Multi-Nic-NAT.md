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

## -> Root Causes Identified

This incident had multiple layered issues:
### 1.Duplicate Gateway IP (Critical)

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
### 2.Host Routing Bypassing VM1

Libvirt was forwarding traffic between networks directly on the host.

Result:

* NAT on VM1 never triggered
* MASQUERADE counters stayed at 0

Fix:
Disabled forwarding in internal network definition.

### 3.NAT Rule Not Matching

Original rule was too generic.

Fix:
Used explicit source matching:
```
iptables -t nat -A POSTROUTING -s 192.168.200.0/24 -o enp1s0 -j MASQUERADE
```

### 4.Reverse Path Filtering Issues

Multiple sysctl files were setting different ```rp_filter``` values.

Strict mode caused asymmetric routing drops.

Fix:
```
net.ipv4.conf.all.rp_filter=0
net.ipv4.conf.default.rp_filter=0
```
### -> Final Working Configuration
## Enable IP Forwarding
```bash
echo "net.ipv4.ip_forward=1" > /etc/sysctl.d/router.conf
sysctl --system
```
### NAT (Router Only)
```bash
iptables -t nat -A POSTROUTING -s 192.168.200.0/24 -o enp1s0 -j MASQUERADE
```
### Libvirt Internal Network (this one of the key area help me to solve the issue)
```XML
<network>
  <name>internal200</name>
  <bridge name='virbr2'/>
  <forward mode='none'/>
</network>
```

## -> What I Learned

- Always debug layer by layer (L2 → L3 → NAT → Hypervisor)
- Never assign duplicate gateway IPs
- Libvirt networks can silently route traffic
- NAT counters tell the truth
- ```tcpdump``` is the most powerful tool in networking
- Reverse path filtering can break multi-NIC routers
- Hypervisor-level networking must be considered in debugging

## Debugging Tools Used

- ```tcpdump```
- ```iptables -t nat -L -v```
- ```ip route```
- ```ip route get```
- ```ip neigh```
- ```brctl show```
- ```virsh net-dumpxml```
- ```sysctl```
