> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 8 — Device Driver Engineering: MMIO, DMA, Queues, and Interrupts](../chapters/08-device-driver-engineering.md)
- [Chapter 13 — Building a Small KVM VMM](../chapters/13-kvm-vmm.md)

---

# Week 44 — Device model

**Objective:** Understand the VMM side of MMIO and device state.

### Reading

- [ ] Review KVM MMIO exits.
- [ ] Review virtqueue ownership and notification.

### Implementation

First implement a simple MMIO register device:

```text
offset 0x00: device ID
offset 0x04: status
offset 0x08: input
offset 0x0c: output
```

Then:

- [ ] validate access width and offset;
- [ ] inject an interrupt;
- [ ] serialize the device state;
- [ ] implement a small descriptor-based queue device;
- [ ] fuzz malformed guest requests.

### Verification

- [ ] Invalid accesses do not corrupt VMM memory.
- [ ] Device reset returns a defined state.
- [ ] Serialized state restores exactly.
- [ ] Guest and VMM agree on ownership transitions.

### Exit artifact

```text
oc-vmm/src/devices/toy_mmio.rs
oc-vmm/src/devices/toy_queue.rs
```

---
