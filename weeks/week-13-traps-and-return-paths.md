> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 2 — The x86-64 Machine Model](../chapters/02-x86-64-machine-model.md)
- [Chapter 5 — OS Interfaces Through xv6](../chapters/05-os-interfaces-xv6.md)

---

# Week 13 — Traps and return paths

**Objective:** Read and modify a real trap entry/exit path.

### Reading

- [ ] xv6 Chapter 4, *Traps and system calls*.
- [ ] Revisit OSTEP Chapter 6.
- [ ] Revisit OSDev exception and ISR pages.

### Implementation

- [ ] Complete the xv6 traps lab.
- [ ] Annotate `trampoline.S` line by line.
- [ ] Add a controlled register dump.
- [ ] Instrument trap causes and return paths.

### Verification

- [ ] You can reconstruct the trap frame from assembly.
- [ ] You can explain which code runs with which page table active.
- [ ] Your annotations are stored in the repository.

### Exit artifact

```text
xv6-labs/week-13/
docs/notes/week-13-traps.md
```

---
