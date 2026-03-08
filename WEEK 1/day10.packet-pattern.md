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


### Pattern 2 – SYN followed by RST
