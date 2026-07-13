> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 13 — Building a Small KVM VMM](../chapters/13-kvm-vmm.md)

---

# Week 43 — VMM event loop and interrupt injection

**Objective:** Build the host-side concurrency model for vCPUs and devices.

### Reading

- [ ] KVM interrupt, irqchip, eventfd, and vCPU-state APIs required by your design.
- [ ] `epoll` and `eventfd` documentation.

### Implementation

- [ ] Run the vCPU in a dedicated thread.
- [ ] Build an `epoll`-based device and control event loop.
- [ ] Use `eventfd` for notifications.
- [ ] Add graceful stop and shutdown.
- [ ] Inject one synthetic interrupt.
- [ ] Record exit count and handling latency by reason.

### Verification

- [ ] Interrupt reaches a guest handler.
- [ ] Stop request reliably terminates the vCPU loop.
- [ ] VM-exit report is generated after every run.

### Exit artifact

```text
oc-vmm/src/vcpu.rs
oc-vmm/src/event_loop.rs
benches/vm-exits/
```

---
