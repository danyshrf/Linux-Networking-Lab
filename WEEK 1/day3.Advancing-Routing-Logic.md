## Day 3: Advanced Routing Logic & Path Selection

**Objective:** Deep dive into Linux routing decisions, prefix matching, metrics, and Equal-Cost Multi-Path (ECMP) routing within a multi-VM environment.

### Core Routing Decision Hierarchy
When Linux routes a packet, it evaluates paths in this strict order:
1. **Connected Routes** (Directly attached networks)
2. **Longest Prefix Match** (The most specific subnet wins, e.g., `/25` beats `/24`)
3. **Lowest Metric** (Tie-breaker for equal prefixes)
4. **ECMP Load Balancing** (If prefix and metric are identical, traffic is hashed per-flow)
5. **Default Route** (`0.0.0.0/0` - used only if no specific match exists)
6. **Drop Packet** (If no match and no default route exists)

### Lab Scenarios Validated
* **Default Route Dependency:** Removed the default route on the primary VM. Verified that directly connected subnets remained reachable, but external traffic was dropped (`Network is unreachable`), proving default gateways are only for remote networks.
* **Multi-Hop Pathing:** Demonstrated that routing is strictly hop-by-hop. Every intermediate router must possess a valid route to the destination; otherwise, packets are dropped mid-transit.
* **Metric vs. Prefix Validation:** Configured overlapping routes to prove that a longer prefix (smaller network) will always beat a lower metric. 
* **ECMP Testing:** Configured dual routes with identical metrics to observe Linux load balancing connections.

### Production Insight: Silent Gateway Failures
Linux **does not** automatically failover if a gateway goes offline silently. If the local interface remains `UP`, Linux will continue routing traffic based on the routing table, resulting in a traffic black hole. True high availability requires dynamic routing protocols or health-checking daemons (like `keepalived`).

### Diagnostic Toolkit
```bash
ip route               # View the active routing table
ip route get 8.8.8.8   # Simulate path selection to see exactly how Linux will route a packet
ip addr                # Verify interface statuses
tcpdump                # Capture packet flow to verify hop-by-hop routing behavior
```

If VM2 has default gateway 192.168.100.1
But VM1 has no default route

Can VM2 reach internet?

->Packet Flow

Scenario:

- VM2 → 192.168.100.10
- Gateway → 192.168.100.1 (VM1)
- VM1 has no default route
- Destination → 8.8.8.8

### Step 1 — VM2 Makes Routing Decision

VM2 checks:

Is 8.8.8.8 inside 192.168.100.0/24?

❌ No.

So it uses default route:
```
default via 192.168.100.1
```
It ARPs for 192.168.100.1
Gets router MAC
Sends packet to VM1

So far everything is correct. 

### Step 2 — VM1 Makes Routing Decision

VM1 receives packet destined to 8.8.8.8

Now VM1 checks:

- 192.168.100.0/24? ❌
- 192.168.200.0/24? ❌
- Any static route? ❌
- Default route? ❌

No match.

So kernel returns:
```
Network unreachable
```
Packet is dropped.

### The Big Rule 

**Every router in the path must have a route to the next hop util it reaches the destination.**

- Networking is hop-by-hop routing.
- There is no “end-to-end global route.”
- Each router makes an independent decision
