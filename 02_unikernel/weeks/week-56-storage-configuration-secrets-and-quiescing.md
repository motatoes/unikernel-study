> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 11 — Block Storage, Persistence, and Checkpoint Semantics](../chapters/11-storage-persistence-snapshots.md)
- [Chapter 7 — Interrupts, Time, Entropy, and Lifecycle State](../chapters/07-interrupts-time-entropy.md)

---

# Week 56 — Storage, configuration, secrets, and quiescing

**Objective:** Add durable data and a safe checkpoint boundary.

### Reading

- [ ] Revisit virtio-block and crash-consistency material.
- [ ] Review Firecracker snapshot guidance.

### Implementation

- [ ] Integrate virtio-block.
- [ ] Attach a separate data device.
- [ ] Implement an append-only key-value service or other bounded durable store.
- [ ] Deliver configuration over vsock.
- [ ] Inject secrets without embedding them in the image.
- [ ] Implement application quiesce.
- [ ] Flush block operations before reporting quiesced state.

### Verification

- [ ] Data persists across clean restart.
- [ ] Snapshot is taken only after quiesce acknowledgement.
- [ ] Secrets are absent from the immutable image and routine logs.
- [ ] Wrong data-volume identity is rejected on restore.

### Exit artifact

```text
oc-uk/apps/kv-service/
docs/QUIESCE-PROTOCOL.md
```

---
