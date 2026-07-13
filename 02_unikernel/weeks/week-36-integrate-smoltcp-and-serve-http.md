> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 10 — Networking From Ethernet to an HTTP Service](../chapters/10-networking.md)

---

# Week 36 — Integrate smoltcp and serve HTTP

**Objective:** Replace the educational stack with a maintained product stack.

### Reading

- [ ] smoltcp documentation, examples, and device traits.
- [ ] OSTEP event-driven concurrency review.

### Implementation

- [ ] Adapt virtio-net to smoltcp device tokens.
- [ ] Configure an IPv4 address statically; add DHCP later if useful.
- [ ] Implement TCP listeners.
- [ ] Implement an HTTP service with:
  - [ ] `GET /health`;
  - [ ] `GET /version`;
  - [ ] `GET /metrics`;
  - [ ] `POST /echo`.
- [ ] Add bounded request size, connection count, and timeouts.

### Verification

- [ ] Sustained TCP requests succeed.
- [ ] Idle and slow clients time out.
- [ ] Memory remains bounded under load.
- [ ] Metrics expose queue, packet, connection, and allocation state.

### Exit artifact

```text
oc-kernel/src/net/smoltcp_adapter.rs
apps/http-service/
```

---
