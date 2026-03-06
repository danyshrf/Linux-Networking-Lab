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
