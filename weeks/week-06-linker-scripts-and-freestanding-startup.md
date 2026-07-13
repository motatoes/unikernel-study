> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 1 — Toolchains, ABIs, ELF, and the Path to `_start`](../chapters/01-toolchain-abi-elf.md)
- [Chapter 6 — Boot, Build, Test, and Debug Bare-Metal Code](../chapters/06-boot-build-debug.md)

---

# Week 06 — Linker scripts and freestanding startup

**Objective:** Control the final image layout rather than accepting compiler defaults.

### Reading

- [ ] CS:APP Chapter 7 review.
- [ ] GNU linker manual: `ENTRY`, `SECTIONS`, output sections, symbols, alignment, `KEEP`, and program headers.
- [ ] OSDev: [Linker Scripts](https://wiki.osdev.org/Linker_Scripts).
- [ ] OSDev: [Limine Bare Bones](https://wiki.osdev.org/Limine_Bare_Bones).

### Implementation

- [ ] Create a freestanding binary with your own `_start`.
- [ ] Write a linker script with distinct `.text`, `.rodata`, `.data`, and `.bss` regions.
- [ ] Export linker-defined symbols for all region boundaries.
- [ ] Clear `.bss` explicitly at startup.
- [ ] Generate a linker map for every build.
- [ ] Add CI validation that rejects writable-executable load segments.

### Verification

- [ ] `readelf -l` shows the expected segment permissions.
- [ ] `.bss` is zeroed before Rust code depends on it.
- [ ] The entry point matches `_start`.
- [ ] The linker map explains every major output section.

### Exit artifact

```text
labs/freestanding-elf/
linker/x86_64.ld
scripts/check-elf-permissions
```

---
