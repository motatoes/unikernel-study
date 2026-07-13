> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 1 — Toolchains, ABIs, ELF, and the Path to `_start`](../chapters/01-toolchain-abi-elf.md)

---

# Week 05 — ELF object files and executable loading

**Objective:** Understand the artifact that your bootloader and VMM will load.

### Reading

- [ ] CS:APP Chapter 7: relocatable files, symbols, relocation, executable files, and static libraries.
- [ ] ELF gABI: file header, program header, section header, symbols, and relocations.
- [ ] OSDev: [ELF](https://wiki.osdev.org/ELF).

### Implementation

- [ ] Write `elf-inspector` without relying on a high-level ELF library for the first version.
- [ ] Print ELF class, endianness, machine, entry point, and header offsets.
- [ ] Print loadable segments, file and memory sizes, alignment, and permissions.
- [ ] Print section names, symbols, and relocations.
- [ ] Detect overlapping or invalid loadable segments.
- [ ] Compare a static C binary, dynamic C binary, and Rust binary.

### Verification

- [ ] Inspector output agrees with `readelf` for selected binaries.
- [ ] Malformed-file tests do not panic or read out of bounds.
- [ ] You can explain sections versus segments.
- [ ] You can explain why `p_memsz` may exceed `p_filesz`.

### Exit artifact

```text
labs/elf-inspector/
docs/notes/week-05-elf.md
```

---
