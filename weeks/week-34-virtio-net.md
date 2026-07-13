> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 9 — Virtio: Transport, Virtqueues, and Device Families](../chapters/09-virtio.md)
- [Chapter 10 — Networking From Ethernet to an HTTP Service](../chapters/10-networking.md)

---

# Week 34 — Virtio-net

**Objective:** Send and receive Ethernet frames through a real virtual NIC.

### Reading

- [ ] Virtio specification: network-device section.
- [ ] OSDev: [Network Stack](https://wiki.osdev.org/Network_Stack).

### Implementation

- [ ] Initialize RX and TX queues.
- [ ] Allocate and recycle packet buffers.
- [ ] Start with polling mode.
- [ ] Add interrupt-driven receive after polling is stable.
- [ ] Connect QEMU to a TAP interface.
- [ ] Add packet, byte, drop, and notification counters.

### Verification

- [ ] Raw Ethernet frames leave and enter the guest.
- [ ] Host packet capture matches guest counters.
- [ ] RX buffer ownership remains valid under load.
- [ ] Device reset does not reuse stale buffers.

### Exit artifact

```text
oc-kernel/src/drivers/virtio/net.rs
scripts/setup-tap
```

---
