> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 5 — OS Interfaces Through xv6](../chapters/05-os-interfaces-xv6.md)
- [Chapter 11 — Block Storage, Persistence, and Checkpoint Semantics](../chapters/11-storage-persistence-snapshots.md)

---

# Week 18 — Filesystem implementation

**Objective:** Understand pathname lookup, inodes, caching, and block writes.

### Reading

- [ ] OSTEP Chapters 39 and 40.
- [ ] xv6 Chapter 10, *File system*.
- [ ] OSDev filesystem overview material as needed.

### Implementation

- [ ] Complete the xv6 filesystem lab.
- [ ] Trace pathname lookup to inode and block access.
- [ ] Instrument buffer-cache hits and misses.
- [ ] Document the write path and locking order.

### Verification

- [ ] You can trace create, write, sync, and close.
- [ ] You can explain inode, directory entry, and file descriptor differences.
- [ ] All official filesystem tests pass.

### Exit artifact

```text
xv6-labs/week-18/
docs/notes/week-18-filesystem.md
```

---
