> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 3 — Physical Memory, Virtual Memory, and Allocation](../chapters/03-memory-management.md)

---

# Week 26 — Inspect and design virtual memory

**Objective:** Understand the bootloader-provided mappings and define your own address-space policy.

### Reading

- [ ] *Writing an OS in Rust*: Introduction to Paging.
- [ ] CS:APP Chapter 9 review.
- [ ] Intel SDM paging sections.
- [ ] OSDev: [Page Tables](https://wiki.osdev.org/Page_Tables).
- [ ] OSDev: [Higher Half Kernel](https://wiki.osdev.org/Higher_Half_Kernel).

### Implementation

- [ ] Walk active page tables.
- [ ] Print page size and permission for each kernel region.
- [ ] Define a virtual-address layout document.
- [ ] Define a direct-physical-map strategy.
- [ ] Reserve regions for heap, stacks, MMIO, and temporary mappings.

### Verification

- [ ] The document explains every major virtual range.
- [ ] Page-table walker matches debugger observations.
- [ ] Null page is identified for removal.

### Exit artifact

```text
docs/architecture-decisions/ADR-0002-virtual-address-layout.md
oc-kernel/src/memory/walk.rs
```

---
