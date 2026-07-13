> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 1 — Toolchains, ABIs, ELF, and the Path to `_start`](../chapters/01-toolchain-abi-elf.md)
- [Chapter 6 — Boot, Build, Test, and Debug Bare-Metal Code](../chapters/06-boot-build-debug.md)

---

# Week 21 — Boot a Rust `no_std` kernel

**Objective:** Reach Rust code on bare-metal x86-64 with a controlled image layout.

### Reading

- [ ] *Writing an OS in Rust*: A Freestanding Rust Binary.
- [ ] *Writing an OS in Rust*: A Minimal Rust Kernel.
- [ ] OSDev: [Boot Sequence](https://wiki.osdev.org/Boot_Sequence).
- [ ] OSDev: [Limine Bare Bones](https://wiki.osdev.org/Limine_Bare_Bones).
- [ ] OSDev: [Higher Half Kernel](https://wiki.osdev.org/Higher_Half_Kernel).

### Implementation

- [ ] Create `oc-kernel` as a `no_std`, `no_main` Rust crate.
- [ ] Boot through Limine.
- [ ] Establish a known stack.
- [ ] Clear `.bss`.
- [ ] Parse the bootloader memory map.
- [ ] Write to COM1 serial.
- [ ] Exit QEMU cleanly.

### Verification

- [ ] Serial output identifies image version and entry address.
- [ ] Memory-map entries are printed and validated.
- [ ] GDB breaks at `_start` and at the Rust entry point.
- [ ] ELF permissions remain RX/R/RW.

### Exit artifact

```text
oc-kernel/boot/
docs/BOOT-ABI-draft.md
```

---
