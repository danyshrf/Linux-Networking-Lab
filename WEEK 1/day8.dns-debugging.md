# Day 9 – DNS Debugging (Name Resolution vs Network Connectivity)

## Goal

Understand how DNS resolution works and how to differentiate **DNS failures** from **network routing failures**.

Focus areas:

- DNS resolution process
- `/etc/resolv.conf` configuration
- Using `dig` to test DNS
- Identifying DNS vs connectivity issues
- Packet-level observation of DNS queries

# 🏗 Lab Architecture

| Machine | IP | Role |
|--------|-------------|------|
| VM1 | 192.168.100.1 | Router |
| VM2 | 192.168.100.10 | Client |

VM2 performs DNS queries to resolve internet domains.

What DNS Does

DNS translates domain names into IP addresses.

Example:
```
google.com → 142.250.x.x
```

Without DNS, systems cannot reach websites using domain names.

However, direct IP communication still works.

Example:

```bash
curl http://142.250.72.206
```
## Step 1 – Verify DNS Works

Check configured DNS servers:
```
cat /etc/resolv.conf
```
Example output:
```
nameserver 8.8.8.8
nameserver 1.1.1.1
```
Test resolution:
```
dig google.com
```
Expected output:
```
ANSWER SECTION:
google.com. 300 IN A 142.250.72.206
```
DNS server successfully returned an IP address.

## Step 2 – Break DNS

Edit resolver configuration:
```
sudo nano /etc/resolv.conf
```
Replace nameserver with an invalid IP:
```
nameserver 10.10.10.10
```

## Step 3 – Test DNS Again

Run:
```
dig google.com
```
Expected result:
```
connection timed out; no servers could be reached
```
Reason:
- The system cannot reach the configured DNS server.

## Test With curl

Try accessing a domain:
```
curl google.com
```

Result:
```
Could not resolve host: google.com
```
The system cannot convert the domain into an IP address.

## Test Direct IP Connectivity

Try bypassing DNS:
```
curl http://142.250.72.206
```
If this works, the network path is working.

This confirms the issue is DNS resolution, not connectivity.

## DNS Failure vs Routing Failure
```
| Test              | DNS Failure | Routing Failure |
| ----------------- | ----------- | --------------- |
| `ping 8.8.8.8`    | Works       | Fails           |
| `curl google.com` | Fails       | Fails           |
| `curl 8.8.8.8`    | Works       | Fails           |
| `dig google.com`  | Fails       | May fail        |
```
Key insight:
- DNS problems affect name resolution, not raw IP communication.

## DNS Debug Commands

Check DNS configuration:
```
cat /etc/resolv.conf
```
Test DNS query:
```
dig google.com
```
Query specific DNS server:
```
dig @8.8.8.8 google.com
```
Alternative DNS test:
```
nslookup google.com
```
## DNS Resolution Order (Linux)

When an application resolves a domain:

- ```/etc/hosts```
- DNS servers from /etc/resolv.conf
- Resolver configuration from /etc/nsswitch.conf
- Query DNS server

Example resolver rule:
```
hosts: files dns
```
Meaning:

- Check ```/etc/hosts```
- Then check DNS servers

## Packet-Level DNS Observation

Capture DNS traffic:
```bash
tcpdump -nn -i enp7s0 port 53
```
Run:
```
dig google.com
```
