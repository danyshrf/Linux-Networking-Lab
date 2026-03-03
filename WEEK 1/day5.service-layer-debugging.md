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

