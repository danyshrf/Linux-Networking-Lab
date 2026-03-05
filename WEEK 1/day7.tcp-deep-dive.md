# TCP Deep Dive (Handshake & Connection Lifecycle)
Understand how TCP connections work at the packet level by capturing and analyzing a real HTTP session.

**Focus areas:**

TCP 3-way handshake

- TCP flags
- Data transfer packets
- TCP connection termination
- Recognizing packet patterns with tcpdump

### Lab Architecture
```
| Machine | IP             | Role               |
| ------- | -------------- | ------------------ |
| VM1     | 192.168.100.1  | Web Server (nginx) |
| VM2     | 192.168.100.10 | Client             |
```
VM2 will send HTTP requests to VM1.

## Step 1 – Start Packet Capture

On VM1 run:Step 1 – Start Packet Capture

On VM1 run:
```bash
tcpdump -nn -i enp7s0 tcp port 80
```
Explanation:
```
| Option        | Meaning                             |
| ------------- | ----------------------------------- |
| `-nn`         | Disable DNS/service name resolution |
| `-i enp7s0`   | Capture packets on this interface   |
| `tcp port 80` | Filter HTTP traffic                 |
```
## Step 2 – Generate Traffic

From VM2:
```bash
curl http://192.168.100.1
```

This triggers a full TCP connection.

## TCP 3-Way Handshake

Example tcpdump output:
```bash
192.168.100.10.45822 > 192.168.100.1.80: Flags [S]
192.168.100.1.80 > 192.168.100.10.45822: Flags [S.]
192.168.100.10.45822 > 192.168.100.1.80: Flags [.]
```

### Step 1 — SYN

Client → Server
```
Flags [S]
```
The client requests a new TCP connection.

### Step 2 — SYN-ACK

Server → Client
```
Flags [S.]
```
The server acknowledges the request and agrees to establish the connection.

### Step 3 — ACK

Client → Server
```
Flags [.]
```
The client confirms the connection.

At this point the TCP connection is ESTABLISHED.

## Data Transfer

After the handshake, HTTP data is exchanged.

Example packet:
```
Flags [P.]
```
Meaning:
```text
| Flag | Meaning                                        |
| ---- | ---------------------------------------------- |
| P    | PUSH – deliver data to application immediately |
| ACK  | Acknowledge received data                      |
```
These packets contain HTTP request/response data.

## TCP Connection Termination

Example capture:
```
Client → Server : FIN
Server → Client : ACK
Server → Client : FIN
Client → Server : ACK
```
This is called TCP 4-way termination.

Why four packets?
- Because TCP is full-duplex, meaning both sides close their data streams independently.

## TCP Flag Reference
| Flag | Meaning                     |
| ---- | --------------------------- |
| SYN  | Start connection            |
| ACK  | Acknowledge packet          |
| PSH  | Push data to application    |
| FIN  | Gracefully close connection |
| RST  | Reset connection            |

## Connection States with ss

Check active TCP connections:
```
ss -tan
```
Example:
```
ESTAB 0 0 192.168.100.10:45822 192.168.100.1:80
```
Meaning:

The connection between client and server is established.

## Packet Pattern Recognition

Packet captures reveal the root cause of many network issues.
```text
| Pattern                        | Meaning                             |
| ------------------------------ | ----------------------------------- |
| SYN → SYN → SYN                | Packet drop or firewall             |
| SYN → RST                      | Port closed / service not listening |
| Handshake succeeds but no data | Application issue                   |
| Duplicate ACK + Retransmission | Packet loss                         |
```
## Debug Scenario Example

Scenario:
```
curl http://192.168.100.1
```
Client hangs.

tcpdump shows:
```
SYN
SYN
SYN
```
Root cause:
```
Firewall DROP rule blocking port 80
```
The SYN-ACK never returns.

## Key Lessons

- TCP uses a 3-way handshake to establish connections.
- Packet capture reveals the exact network behavior.
- SYN / SYN-ACK / ACK confirms connection establishment.
- TCP flags indicate connection state.
- Packet pattern recognition is critical for network troubleshooting.

## Why Using 

Packet analysis is used to debug real incidents such as:

- Web servers not responding
- Firewall misconfiguration
- Packet loss in networks
- Application timeouts
- TCP retransmission issues
- Tools like tcpdump provide the ground truth of network behavior.
