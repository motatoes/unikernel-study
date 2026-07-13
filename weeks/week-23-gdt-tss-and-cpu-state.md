> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 2 — The x86-64 Machine Model](../chapters/02-x86-64-machine-model.md)

---

# Week 23 — GDT, TSS, and CPU state

**Objective:** Establish architectural tables and dedicated fault stacks.

### Reading

- [ ] Intel SDM: segmentation in long mode, descriptor tables, TSS, and IST.
- [ ] OSDev: [Global Descriptor Table](https://wiki.osdev.org/Global_Descriptor_Table).

### Implementation

- [ ] Define and load a GDT.
- [ ] Define a TSS.
- [ ] Allocate a dedicated interrupt stack.
- [ ] Validate segment selectors.
- [ ] Dump relevant CPU state at boot.

### Verification

- [ ] GDT and TSS load successfully.
- [ ] Dedicated stack address is in a valid mapped range.
- [ ] Invalid selector experiments are isolated and documented.

### Exit artifact

```text
oc-kernel/src/arch/x86_64/gdt.rs
oc-kernel/src/arch/x86_64/tss.rs
```

---
