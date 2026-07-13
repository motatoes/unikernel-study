> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 9 — Virtio: Transport, Virtqueues, and Device Families](../chapters/09-virtio.md)

---

# Week 33 — Virtio-MMIO transport

**Objective:** Negotiate and configure a device over the MMIO transport.

### Reading

- [ ] Virtio specification: MMIO transport section.
- [ ] Device initialization and status-state requirements.

### Implementation

- [ ] Discover a virtio-MMIO device.
- [ ] Validate magic, version, device ID, and vendor ID.
- [ ] Negotiate feature bits.
- [ ] Configure queue addresses and size.
- [ ] Drive the status state machine correctly.
- [ ] Handle device failure and reset.
- [ ] Communicate with a toy queue-based device before virtio-net.

### Verification

- [ ] Invalid state transitions are rejected.
- [ ] Unsupported required features fail cleanly.
- [ ] Reset returns the driver to a known state.
- [ ] Toy device transfers a byte buffer in both directions.

### Exit artifact

```text
oc-kernel/src/drivers/virtio/mmio.rs
oc-kernel/tests/virtio-mmio.rs
```

---
