> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 5 — OS Interfaces Through xv6](../chapters/05-os-interfaces-xv6.md)

---

# Week 09 — Processes and the Unix interface

**Objective:** Understand the abstractions your unikernel will intentionally remove.

### Reading

- [ ] OSTEP Chapters 2, 4, and 5.
- [ ] xv6 Chapter 1, *Operating system interfaces*.
- [ ] OSDev overview of [System Calls](https://wiki.osdev.org/System_Calls).

### Implementation

- [ ] Set up the xv6 tree and toolchain.
- [ ] Complete the Unix utilities lab.
- [ ] Trace one command from user entry through one system call.
- [ ] Record process creation, execution, waiting, and exit.

### Verification

- [ ] You can explain the purpose of `fork`, `exec`, `wait`, and file descriptors.
- [ ] You can identify which process abstractions the first unikernel will omit.
- [ ] Your xv6 changes pass the official tests.

### Exit artifact

```text
xv6-labs/week-09/
docs/notes/week-09-process-interface.md
```

---
