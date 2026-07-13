> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 5 — OS Interfaces Through xv6](../chapters/05-os-interfaces-xv6.md)

---

# Week 20 — xv6 capstone and OS model review

**Objective:** Demonstrate that you can modify a complete kernel subsystem and defend the design.

### Reading

- [ ] xv6 Chapter 11, *Concurrency revisited*.
- [ ] Review relevant OSTEP chapters for your capstone.

### Implementation

Choose one:

- [ ] priority or lottery scheduler;
- [ ] enhanced copy-on-write;
- [ ] network-driver enhancement;
- [ ] tracing subsystem;
- [ ] memory-mapped-file enhancement;
- [ ] filesystem consistency feature.

Write a design report covering:

- [ ] invariants;
- [ ] locking;
- [ ] failure cases;
- [ ] performance impact;
- [ ] tests;
- [ ] what changes on x86-64;
- [ ] which abstractions the unikernel can remove.

### Verification

- [ ] The capstone passes automated tests.
- [ ] The report includes a complete path diagram.
- [ ] You can defend the design in a recorded walkthrough or written review.

### Exit artifact

```text
xv6-labs/capstone/
docs/xv6-capstone-design.md
```

## Phase 2 gate

- [ ] Completed xv6 system-call, page-table, traps, COW, network, lock, filesystem, and mmap work.
- [ ] Can trace system calls, faults, scheduling, packets, and writes end to end.
- [ ] Can explain which general-purpose OS abstractions the unikernel will omit.
- [ ] Completed one substantial xv6 capstone.

---
