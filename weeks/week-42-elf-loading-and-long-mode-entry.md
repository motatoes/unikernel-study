> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 13 — Building a Small KVM VMM](../chapters/13-kvm-vmm.md)

---

# Week 42 — ELF loading and long-mode entry

**Objective:** Boot your own x86-64 image without QEMU.

### Reading

- [ ] KVM x86 register and special-register APIs.
- [ ] Review ELF loading and page-table setup.

### Implementation

- [ ] Parse your kernel ELF.
- [ ] Copy loadable segments into guest memory.
- [ ] Zero segment memory beyond file size.
- [ ] Create initial page tables.
- [ ] Initialize control registers and long-mode state.
- [ ] Set RIP, RSP, and boot-information pointer.
- [ ] Capture serial-like output.

### Verification

- [ ] `oc-vmm` reaches the same kernel entry as QEMU.
- [ ] Segment permissions and addresses match ELF metadata.
- [ ] Invalid ELF images are rejected before vCPU execution.

### Exit artifact

```text
oc-vmm/src/elf_loader.rs
oc-vmm/src/x86_64/boot.rs
```

---
