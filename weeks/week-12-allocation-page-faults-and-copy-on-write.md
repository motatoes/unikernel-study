> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 3 — Physical Memory, Virtual Memory, and Allocation](../chapters/03-memory-management.md)
- [Chapter 5 — OS Interfaces Through xv6](../chapters/05-os-interfaces-xv6.md)

---

# Week 12 — Allocation, page faults, and copy-on-write

**Objective:** Make demand allocation and copy-on-write concrete.

### Reading

- [ ] OSTEP Chapters 17–20.
- [ ] xv6 Chapter 5, *Page faults*.
- [ ] OSDev: [Page Frame Allocation](https://wiki.osdev.org/Page_Frame_Allocation).

### Implementation

- [ ] Complete the xv6 copy-on-write lab.
- [ ] Count page faults by cause.
- [ ] Count allocations, reference-count changes, and copied pages.
- [ ] Add tests for writes to shared pages and process exit.

### Verification

- [ ] No leaked shared frames after process exit.
- [ ] Writes cause copies only where required.
- [ ] You can explain the TLB consequences of changing a PTE.

### Exit artifact

```text
xv6-labs/week-12/
docs/notes/week-12-cow.md
```

---
