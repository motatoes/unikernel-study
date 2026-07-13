> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 8 — Device Driver Engineering: MMIO, DMA, Queues, and Interrupts](../chapters/08-device-driver-engineering.md)

---

# Week 31 — Device interfaces, MMIO, and DMA contracts

**Objective:** Define driver-facing abstractions before implementing a particular virtio device.

### Reading

- [ ] OSTEP Chapter 36 review.
- [ ] Intel memory-ordering and cacheability material relevant to MMIO/DMA.
- [ ] OSDev: [PCI](https://wiki.osdev.org/PCI).
- [ ] OSDev: [PCI Express](https://wiki.osdev.org/PCI_Express).
- [ ] OSDev: [Virtio](https://wiki.osdev.org/Virtio).

### Implementation

- [ ] Define typed MMIO register access.
- [ ] Define DMA region ownership and address translation.
- [ ] Define `NetDevice`, `BlockDevice`, and generic event-source traits.
- [ ] Build a fake MMIO device for hosted tests.
- [ ] Document reset and failure behaviour.

### Verification

- [ ] Host tests can emulate reads, writes, reset, and interrupts.
- [ ] Driver traits do not expose QEMU-specific types.
- [ ] Every register field has width, access, and state semantics documented.

### Exit artifact

```text
oc-kernel/src/drivers/traits.rs
labs/fake-mmio-device/
```

---
