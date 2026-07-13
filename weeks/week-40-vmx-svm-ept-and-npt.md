> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 12 — Virtualization Theory: From Trap-and-Emulate to KVM](../chapters/12-virtualization-theory.md)

---

# Week 40 — VMX, SVM, EPT, and NPT

**Objective:** Understand the hardware mechanisms KVM exposes without implementing them directly.

### Reading

- [ ] Intel SDM VMX overview, VM entry/exit, VMCS, and EPT sections.
- [ ] AMD architecture material on SVM, VMCB, and NPT.

### Implementation

- [ ] Draw VMX root/non-root transitions.
- [ ] Draw a VM-exit handling sequence.
- [ ] Compare VMCS and VMCB.
- [ ] Compare EPT and NPT.
- [ ] Map common exit reasons to VMM responsibilities.

### Verification

- [ ] You can explain why a guest page fault differs from an EPT/NPT violation.
- [ ] You can explain CPUID filtering, MSR handling, and interrupt injection conceptually.
- [ ] You can identify which work KVM performs in the kernel and which remains in userspace.

### Exit artifact

```text
docs/notes/week-40-hardware-virtualization.md
```

---
