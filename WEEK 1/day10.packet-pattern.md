## Packet Pattern Recognition (tcpdump)

When debugging connectivity issues, packet patterns reveal the root cause quickly.

### Pattern 1 – SYN Retransmissions

```
SYN
SYN
SYN
```

Meaning:

Client retries connection but server never responds.

Possible causes:

- firewall DROP
- routing issue
- host unreachable

Example:
```
tcpdump -nn port 80
```
Output:
```
192.168.100.10 > 192.168.100.1 SYN
192.168.100.10 > 192.168.100.1 SYN
192.168.100.10 > 192.168.100.1 SYN
```
Mental model:
```
Client knocking on door
No one answering
```

### Pattern 2 – SYN followed by RST

Packet capture:
```
SYN
RST
```
Meaning:
```
Host reachable
But port closed
```
Typical causes:

- service stopped
- wrong port
- application not listening

Example:
```
systemctl stop nginx
```
Then run:
```
curl server-ip
```

Packet pattern:
```
SYN
RST
```
Mental model:
```
Door exists
But nobody is running the service
```
