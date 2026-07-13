> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 3 — Physical Memory, Virtual Memory, and Allocation](../chapters/03-memory-management.md)
- [Chapter 5 — OS Interfaces Through xv6](../chapters/05-os-interfaces-xv6.md)

---

# Week 11 — Address spaces and page tables

**Objective:** Understand page-table structure and kernel/user mappings through a working OS.

### Reading

- [ ] OSTEP Chapters 13–15.
- [ ] xv6 Chapter 3, *Page tables*.
- [ ] OSDev: [Page Tables](https://wiki.osdev.org/Page_Tables).

### Implementation

- [ ] Complete the xv6 page-table lab.
- [ ] Add a page-table dump with permissions and physical targets.
- [ ] Trace one user virtual address through every page-table level.
- [ ] Identify kernel global mappings and per-process mappings.

### Verification

- [ ] You can explain PTE permission bits and address translation.
- [ ] You can distinguish a virtual address, physical address, and page-frame number.
- [ ] Dumped mappings match expected process layout.

### Exit artifact

```text
xv6-labs/week-11/
docs/notes/week-11-page-tables.md
```

---
