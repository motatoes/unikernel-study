> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 1 — Toolchains, ABIs, ELF, and the Path to `_start`](../chapters/01-toolchain-abi-elf.md)
- [Chapter 3 — Physical Memory, Virtual Memory, and Allocation](../chapters/03-memory-management.md)

---

# Week 03 — Data representation and typed addresses

**Objective:** Make machine representation, layout, alignment, and address domains explicit.

### Reading

- [ ] CS:APP Chapter 2, *Representing and Manipulating Information*.
- [ ] Rust Book Chapters 4 and 10.
- [ ] Rustonomicon: data layout and unsafe foundations.
- [ ] OSDev: [Rust](https://wiki.osdev.org/Rust).
- [ ] OSDev: [System V ABI](https://wiki.osdev.org/System_V_ABI).

### Implementation

- [ ] Implement `PhysAddr`, `VirtAddr`, `GuestPhysAddr`, and `GuestVirtAddr` newtypes.
- [ ] Implement checked alignment helpers: `align_up`, `align_down`, and `is_aligned`.
- [ ] Implement endian wrapper types for little- and big-endian integers.
- [ ] Implement narrow `volatile_read` and `volatile_write` APIs.
- [ ] Define an aligned DMA buffer wrapper with a documented ownership model.
- [ ] Inspect `#[repr(C)]`, `#[repr(transparent)]`, and packed-layout behaviour.

### Verification

- [ ] Tests cover overflow, canonical addresses, alignment, and endian conversion.
- [ ] No public API uses a naked `usize` where the address domain matters.
- [ ] Every unsafe function contains a `# Safety` contract.
- [ ] You can explain padding and alignment in one device-register struct.

### Exit artifact

```text
labs/address-types/
docs/notes/week-03-data-layout.md
```

---
