> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 9 — Virtio: Transport, Virtqueues, and Device Families](../chapters/09-virtio.md)

---

# Week 32 — Split virtqueue core

**Objective:** Implement the transport-independent queue data structure.

### Reading

- [ ] Virtio specification: basic facilities, device status, feature negotiation, and split virtqueues.
- [ ] OSDev: [Virtio](https://wiki.osdev.org/Virtio).

### Implementation

- [ ] Implement descriptor allocation and release.
- [ ] Implement direct and chained descriptors.
- [ ] Implement available and used rings.
- [ ] Handle index wrap-around.
- [ ] Define ownership transitions.
- [ ] Add memory barriers at publication and completion points.
- [ ] Implement queue reset.

### Verification

- [ ] No descriptor is returned twice.
- [ ] Descriptor chains cannot loop indefinitely.
- [ ] Out-of-range descriptor references are rejected.
- [ ] Ring wrap-around tests pass.
- [ ] Broken-ordering tests demonstrate why barriers matter.

### Exit artifact

```text
oc-kernel/src/drivers/virtio/queue.rs
oc-kernel/tests/virtqueue.rs
```

---
