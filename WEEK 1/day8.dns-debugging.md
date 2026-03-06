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
