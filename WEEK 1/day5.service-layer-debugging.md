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
