> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 12 — Virtualization Theory: From Trap-and-Emulate to KVM](../chapters/12-virtualization-theory.md)

---

# Week 39 — Virtualization models

**Objective:** Understand the major ways systems virtualize privileged execution.

### Reading

- [ ] Popek and Goldberg.
- [ ] *Xen and the Art of Virtualization*.
- [ ] *KVM: the Linux Virtual Machine Monitor*.
- [ ] Optional: *Virtual Machines*, Chapters 1 and 8.

### Implementation

- [ ] Write a comparison of trap-and-emulate, binary translation, paravirtualization, and hardware-assisted virtualization.
- [ ] Draw the protection boundaries in Xen, KVM/QEMU, and Firecracker.
- [ ] Identify where device emulation runs in each design.

### Verification

- [ ] You can explain why classic x86 was difficult to virtualize.
- [ ] You can distinguish guest virtual, guest physical, and host virtual addresses.
- [ ] You can explain why virtio is paravirtualized even when the CPU is hardware-virtualized.

### Exit artifact

```text
docs/notes/week-39-virtualization-models.md
```

---
