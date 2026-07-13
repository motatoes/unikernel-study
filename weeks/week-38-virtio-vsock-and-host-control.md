> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 9 — Virtio: Transport, Virtqueues, and Device Families](../chapters/09-virtio.md)
- [Chapter 19 — Targeting Firecracker](../chapters/19-firecracker-target.md)

---

# Week 38 — Virtio-vsock and host control

**Objective:** Add a narrow control path independent of the guest network interface.

### Reading

- [ ] Virtio specification: socket-device section.
- [ ] Firecracker [vsock documentation](https://github.com/firecracker-microvm/firecracker/blob/main/docs/vsock.md).

### Implementation

- [ ] Implement or integrate virtio-vsock.
- [ ] Define a versioned control protocol.
- [ ] Support configuration delivery.
- [ ] Support readiness notification.
- [ ] Support structured logs or log-control messages.
- [ ] Support quiesce and shutdown requests.
- [ ] Bound message size and connection count.

### Verification

- [ ] Host can deliver configuration before application readiness.
- [ ] Guest sends a machine-readable ready event.
- [ ] Invalid control messages do not crash the guest.
- [ ] Shutdown completes within a bounded timeout.

### Exit artifact

```text
oc-kernel/src/drivers/virtio/vsock.rs
docs/HOST-CONTROL-PROTOCOL.md
```

## Phase 4 gate

- [ ] Generic split virtqueue.
- [ ] Virtio-MMIO transport.
- [ ] Virtio-net with TAP integration.
- [ ] Educational UDP stack and fuzzed parsers.
- [ ] smoltcp TCP/HTTP service.
- [ ] Virtio-block with persistence tests.
- [ ] Virtio-vsock control protocol.

---
