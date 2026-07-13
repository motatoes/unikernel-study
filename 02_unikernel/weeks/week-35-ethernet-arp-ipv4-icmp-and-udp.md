> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 10 — Networking From Ethernet to an HTTP Service](../chapters/10-networking.md)

---

# Week 35 — Ethernet, ARP, IPv4, ICMP, and UDP

**Objective:** Learn packet formats and checksums by implementing a small educational stack.

### Reading

- [ ] OSDev: [Ethernet](https://wiki.osdev.org/Ethernet).
- [ ] OSDev: [ARP](https://wiki.osdev.org/Address_Resolution_Protocol).
- [ ] OSDev: [Internet Protocol](https://wiki.osdev.org/Internet_Protocol).
- [ ] OSDev: [ICMP](https://wiki.osdev.org/Internet_Control_Message_Protocol).
- [ ] OSDev: [UDP](https://wiki.osdev.org/User_Datagram_Protocol).
- [ ] Relevant RFCs.

### Implementation

- [ ] Parse and generate Ethernet II frames.
- [ ] Implement ARP request/reply and a bounded cache.
- [ ] Validate IPv4 version, header length, total length, and checksum.
- [ ] Implement ICMP echo request/reply.
- [ ] Implement UDP parsing, checksum policy, and echo service.
- [ ] Fuzz each parser in hosted mode.

### Verification

- [ ] Host can ping the guest.
- [ ] Host can use UDP echo.
- [ ] Truncated and malformed packets are rejected safely.
- [ ] ARP cache has bounded memory and expiry behaviour.

### Exit artifact

```text
labs/network-protocol-lab/
oc-kernel/src/net/educational/
fuzz/network-parsers/
```

---
