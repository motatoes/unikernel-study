> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 3 — Physical Memory, Virtual Memory, and Allocation](../chapters/03-memory-management.md)
- [Chapter 11 — Block Storage, Persistence, and Checkpoint Semantics](../chapters/11-storage-persistence-snapshots.md)

---

# Week 19 — Crash consistency and memory mapping

**Objective:** Learn the difference between memory correctness and durable-state correctness.

### Reading

- [ ] OSTEP Chapter 42, *Crash Consistency: FSCK and Journaling*.
- [ ] xv6 material required for the `mmap` lab.

### Implementation

- [ ] Complete the xv6 `mmap` lab.
- [ ] Inject simulated crashes around filesystem updates.
- [ ] Record which invariants survive.
- [ ] Compare filesystem freeze, journal commit, and application-level quiescing.

### Verification

- [ ] Mappings are released correctly on exit.
- [ ] Protection errors are handled correctly.
- [ ] Crash experiments have a written expected/observed matrix.

### Exit artifact

```text
xv6-labs/week-19/
docs/notes/week-19-crash-consistency.md
```

---
