## Day 5: Service Layer Debugging (nginx, Firewalls & Binding)

**Objective:** Differentiate between application-layer failures caused by service state, firewall rules, and port binding misconfigurations.

Focus:

- nginx service state
- Port listening (ss -tulnp)
- DROP vs REJECT
- Bind address behavior
- Client-side error interpretation

###  Lab Architecture
| Machine | IP | Role |
| :--- | :--- | :--- |
| **VM1** | `192.168.100.1` | Router + Web Server (nginx) |
| **VM2** | `192.168.100.10` | Client |


## Baseline Setup

On VM1:
```bash
dnf install nginx -y
systemctl start nginx
systemctl enable nginx
```
Verify listening:
```bash
ss -tulnp | grep :80
```
Expected:
```bash
LISTEN 0 511 0.0.0.0:80
```
From VM2:
```bash
curl http://192.168.100.1
```
Success.

## Failure Scenario 1 – Service Stopped

On VM1:
```
systemctl stop nginx
```
Test from VM2:
```
curl http://192.168.100.1
```
Result:
```
Connection refused
```
### Why?

- Packet reaches VM1
- No process listening on port 80
- Kernel sends TCP RST

Verification:
```
ss -tulnp | grep :80
```
No output.

## Failure Scenario 2 – Firewall DROP Rule

On VM1:
```bash
iptables -A INPUT -p tcp --dport 80 -j DROP
```
nginx still running.

From VM2:
```
curl http://192.168.100.1
```
Result:
```
Connection timed out
```
### Why?

- Packet reaches VM1
- Firewall silently drops it
- No response sent back
- Client waits until timeout

Firewall rule meaning:
```text
| Option     | Meaning                   |
| ---------- | ------------------------- |
| -A INPUT   | Append to INPUT chain     |
| -p tcp     | Match TCP traffic         |
| --dport 80 | Match destination port 80 |
| -j DROP    | Silently discard packet   |
```

## Failure Scenario 3 – Firewall REJECT Rule

On VM1:
```bash
iptables -A INPUT -p tcp --dport 80 -j REJECT
```
From VM2:
```
curl http://192.168.100.1
```
Result:
```
Connection refused
or
No route to host
```
### Why?

- Firewall actively sends error response
- Client fails immediately

Difference from DROP:
```
| DROP    | REJECT            |
| ------- | ----------------- |
| Silent  | Sends error       |
| Timeout | Immediate failure |
```

## Failure Scenario 4 – Bind to Loopback Only

Edit nginx config:
```code
listen 127.0.0.1:80;
```
Restart nginx:
```
systemctl restart nginx
```
Check:
```
ss -tulnp | grep :80
```
Output:
```
127.0.0.1:80
```
From VM2:
```bash
curl http://192.168.100.1
```
Result:
```code
Connection refused
```
### Why?

- nginx is listening only on loopback
- No service bound to 192.168.100.1
- Kernel sends RST


## Error Messages and their cause
```text
| Client Error         | Root Cause                           |
| -------------------- | ------------------------------------ |
| Connection refused   | Service not listening OR REJECT rule |
| Connection timed out | Firewall DROP                        |
| No route to host     | ICMP reject or routing issue         |
| Network unreachable  | No route in routing table            |
```

##Layer Separation Learned

Routing decides:
- Where should packet go?

Firewall decides:
- Is packet allowed?

Service decides:
- Is process listening?

Bind address decides:
- On which interface is service available?

### Debug Commands Used
```Bash
ss -tulnp
iptables -L -n -v
systemctl status nginx
curl
ping
```

## Production Relevance

- This lab simulates common real-world tickets:
- “Web server is down”
- “Port is not reachable”
- “Ping works but site doesn’t”
- “Timeout vs refused confusion”
