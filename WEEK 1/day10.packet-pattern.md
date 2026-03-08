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
### Pattern 2 - SYN → SYN-ACK → ACK → RST

Packet capture:
```
SYN
SYN-ACK
ACK
RST
```
Meaning:
```
Connection established
Then immediately aborted
```
Typical causes:

- client application closed socket
- protocol mismatch
- TLS/HTTP mismatch
- application logic rejecting connection

Mental model:
```
Connection accepted
Then immediately rejected
```

### Pattern 4 - SYN → SYN-ACK → ACK (Handshake OK but request hangs)

Packet capture:
```
SYN
SYN-ACK
ACK
PSH
ACK
```
But client waits forever.

Meaning:
- Network works
- Application broken

Typical causes:

- backend service stuck
- database query hanging
- slow API

Mental model:
```
Connection established
Server not responding properly
```
### The Packet Pattern Cheat Sheet
| Pattern               | Root Cause               |
| --------------------- | ------------------------ |
| SYN SYN SYN           | Firewall / routing issue |
| SYN RST               | Port closed              |
| SYN SYN-ACK ACK RST   | Connection aborted       |
| Handshake OK but hang | Application issue        |


## Packet Pattern
```bash
SYN
SYN-ACK
ACK
PSH
ACK
FIN
ACK
FIN
ACK
```
This shows three phases of a TCP connection:

1. Connection establishment
2. Data transfer
3. Graceful termination

1. Connection Establishment (3-way handshake)
```
SYN
SYN-ACK
ACK
```
Meaning:
| Step | Packet  | Explanation                     |
| ---- | ------- | ------------------------------- |
| 1    | SYN     | Client asks to start connection |
| 2    | SYN-ACK | Server accepts                  |
| 3    | ACK     | Client confirms                 |

Now the connection state becomes:
```
ESTABLISHED
```

2. Data Transfer
```
PSH
ACK
```
PSH means:
```
Send this data immediately to the application
```
Typical example:
```
HTTP request / response
```
Example flow:
```
Client → HTTP request
Server → HTTP response
```

3. Graceful Connection Termination
```
FIN
ACK
FIN
ACK
```
This is the 4-way TCP close.

Why four packets?

Because TCP is full duplex.

Each side closes its own direction separately.
| Step | Packet | Meaning                 |
| ---- | ------ | ----------------------- |
| 1    | FIN    | Client finished sending |
| 2    | ACK    | Server acknowledges     |
| 3    | FIN    | Server finished sending |
| 4    | ACK    | Client acknowledges     |

Connection state becomes:
```
CLOSED
```
### Visual Lifecycle
```text
Client                     Server
   | SYN ------------------>  |
   | <--------------- SYN-ACK |
   | ACK ------------------>  |
   |                          |
   | PSH (data) ---------->   |
   | <------------- ACK       |
   |                          |
   | FIN -------------------> |
   | <------------- ACK       |
   | <------------- FIN       |
   | ACK ------------------>  |
```
Clean connection lifecycle.

### If you see this pattern in tcpdump:
```
SYN → SYN-ACK → ACK → data → FIN
```
Then you immediately know:
```
✔ Routing works
✔ Firewall allows traffic
✔ Service running
✔ Application responded
✔ Connection closed properly
```
This is a healthy connection.
