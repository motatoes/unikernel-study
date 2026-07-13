> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 1 — Toolchains, ABIs, ELF, and the Path to `_start`](../chapters/01-toolchain-abi-elf.md)
- [Chapter 2 — The x86-64 Machine Model](../chapters/02-x86-64-machine-model.md)

---

# Week 04 — x86-64 assembly and calling conventions

**Objective:** Cross language boundaries without guessing about stack or register state.

### Reading

- [ ] CS:APP Chapter 3, *Machine-Level Representation of Programs*.
- [ ] x86-64 psABI: function calling sequence, register classes, stack alignment, and process entry.
- [ ] Rust Reference: inline assembly.

### Implementation

- [ ] Write an assembly `_start` that calls Rust.
- [ ] Call assembly functions from Rust and Rust functions from assembly.
- [ ] Save and restore all callee-saved registers.
- [ ] Verify stack alignment before a function call.
- [ ] Implement a host-side `CpuContext` structure.
- [ ] Write one deliberately broken example that violates the ABI and document the symptom.

### Verification

- [ ] Disassembly matches your expected parameter and return-value placement.
- [ ] Stack alignment tests pass.
- [ ] Register-preservation tests pass.
- [ ] You can annotate every instruction in your startup stub.

### Exit artifact

```text
labs/abi-lab/
docs/notes/week-04-x86-64-abi.md
```

---
