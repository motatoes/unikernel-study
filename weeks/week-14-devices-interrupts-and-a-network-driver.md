> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 5 — OS Interfaces Through xv6](../chapters/05-os-interfaces-xv6.md)
- [Chapter 8 — Device Driver Engineering: MMIO, DMA, Queues, and Interrupts](../chapters/08-device-driver-engineering.md)

---

# Week 14 — Devices, interrupts, and a network driver

**Objective:** Trace hardware-visible queue state into kernel network processing.

### Reading

- [ ] OSTEP Chapter 36, *I/O Devices*.
- [ ] xv6 Chapter 6, *Interrupts and device drivers*.
- [ ] OSDev: [Interrupts Tutorial](https://wiki.osdev.org/Interrupts_Tutorial).
- [ ] OSDev: [Network Stack](https://wiki.osdev.org/Network_Stack).

### Implementation

- [ ] Begin or complete the xv6 network-driver lab.
- [ ] Instrument descriptor ownership and ring indices.
- [ ] Correlate guest driver logs with host packet captures.
- [ ] Count interrupts, transmitted packets, received packets, and drops.

### Verification

- [ ] You can trace a packet from host interface to guest receiver.
- [ ] Ring ownership does not become ambiguous.
- [ ] Driver counters agree with packet capture within expected limits.

### Exit artifact

```text
xv6-labs/week-14/
docs/notes/week-14-driver-path.md
```

---
