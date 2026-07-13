> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 9 — Virtio: Transport, Virtqueues, and Device Families](../chapters/09-virtio.md)
- [Chapter 11 — Block Storage, Persistence, and Checkpoint Semantics](../chapters/11-storage-persistence-snapshots.md)

---

# Week 37 — Virtio-block and simple durable state

**Objective:** Learn block request semantics without implementing a large filesystem.

### Reading

- [ ] Virtio specification: block-device section.
- [ ] OSTEP Chapters 40 and 42 review.
- [ ] OSDev storage/filesystem material as needed.

### Implementation

- [ ] Implement block reads.
- [ ] Implement block writes.
- [ ] Implement flush.
- [ ] Handle read-only devices.
- [ ] Add either:
  - [ ] a read-only archive over block storage; or
  - [ ] an append-only key-value log.
- [ ] Define crash and partial-failure semantics.

### Verification

- [ ] Reads and writes survive a guest restart where expected.
- [ ] Flush is observable in a controlled test.
- [ ] Read-only attachment rejects writes.
- [ ] Out-of-range requests fail safely.

### Exit artifact

```text
oc-kernel/src/drivers/virtio/block.rs
apps/kv-service/
```

---
