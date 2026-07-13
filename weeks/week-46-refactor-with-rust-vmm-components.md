> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 13 — Building a Small KVM VMM](../chapters/13-kvm-vmm.md)

---

# Week 46 — Refactor with rust-vmm components

**Objective:** Compare first-principles understanding with reusable production-oriented components.

### Reading

- [ ] rust-vmm community and selected crates.
- [ ] Documentation for `kvm-ioctls`, `vm-memory`, and any virtio crates you use.

### Implementation

- [ ] Replace selected raw ioctl wrappers with `kvm-ioctls`.
- [ ] Replace guest-memory boilerplate with `vm-memory` where useful.
- [ ] Preserve your own architecture boundaries.
- [ ] Benchmark code size, complexity, and runtime behaviour before and after.

### Verification

- [ ] All previous VMM tests still pass.
- [ ] The refactor reduces boilerplate without obscuring lifecycle invariants.
- [ ] The report identifies which components you would use in production.

### Exit artifact

```text
docs/notes/week-46-rust-vmm-evaluation.md
```

## Phase 5 gate

- [ ] Raw KVM VM and vCPU creation.
- [ ] Real-mode guest execution.
- [ ] x86-64 ELF loading and long-mode entry.
- [ ] Event loop and interrupt injection.
- [ ] Toy MMIO and queue devices.
- [ ] Stopped-state save and restore.
- [ ] rust-vmm evaluation.

---
