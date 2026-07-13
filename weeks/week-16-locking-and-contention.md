> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 4 — Concurrency, Scheduling, and Event-Driven Execution](../chapters/04-concurrency-execution.md)
- [Chapter 5 — OS Interfaces Through xv6](../chapters/05-os-interfaces-xv6.md)

---

# Week 16 — Locking and contention

**Objective:** Identify kernel invariants protected by locks and measure contention.

### Reading

- [ ] OSTEP Chapters 26–30.
- [ ] xv6 Chapter 7, *Locking*.
- [ ] OSDev: [Spinlock](https://wiki.osdev.org/Spinlock).

### Implementation

- [ ] Complete the xv6 lock lab.
- [ ] Measure contention in allocator and buffer-cache locks.
- [ ] Add lock-order documentation.
- [ ] Create one forced deadlock or lock-order violation test in an isolated branch.

### Verification

- [ ] Lock contention is measurably reduced where the lab requires it.
- [ ] Every modified lock protects an explicit invariant.
- [ ] You can explain interrupt disabling around selected lock paths.

### Exit artifact

```text
xv6-labs/week-16/
docs/notes/week-16-locking.md
```

---
