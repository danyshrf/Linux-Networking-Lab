# TCP Deep Dive (Handshake & Connection Lifecycle)

 ## Goal
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

```
