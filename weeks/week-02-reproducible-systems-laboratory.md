> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 0 — How to Use This Handbook](../chapters/00-orientation.md)
- [Chapter 6 — Boot, Build, Test, and Debug Bare-Metal Code](../chapters/06-boot-build-debug.md)

---

# Week 02 — Reproducible systems laboratory

**Objective:** Build a repeatable environment for compiling, booting, debugging, testing, and measuring systems code.

### Reading

- [ ] OSDev: [Testing](https://wiki.osdev.org/Testing).
- [ ] OSDev: [What Order Should I Make Things In?](https://wiki.osdev.org/What_order_should_I_make_things_in).
- [ ] QEMU: [GDB usage](https://www.qemu.org/docs/master/system/gdb.html).
- [ ] Nix language and flakes material required to define a development shell.

### Implementation

- [ ] Create `flake.nix` with Rust, LLVM, Clang, LLD, binutils, NASM, QEMU, GDB, Python, `tcpdump`, `socat`, and `just`.
- [ ] Add `just build`, `run`, `debug`, `test`, `inspect`, and `clean` targets.
- [ ] Create a script that starts QEMU with serial output captured to a file.
- [ ] Create a script that starts QEMU with `-s -S` and attaches GDB.
- [ ] Create a machine-readable success/failure convention for VM tests.
- [ ] Record baseline QEMU process startup and minimal Linux guest boot time.

### Verification

- [ ] A clean clone enters the same development shell.
- [ ] QEMU starts paused and GDB attaches before guest execution.
- [ ] Serial output is captured in CI-friendly form.
- [ ] Your baseline measurements are stored as JSON or CSV.

### Exit artifact

```text
flake.nix
justfile
scripts/run-qemu
scripts/debug-qemu
scripts/measure-boot
baselines/linux-qemu.json
```

## Phase 0 gate

- [ ] Reproducible development environment.
- [ ] Repeatable QEMU execution.
- [ ] GDB attach before first guest instruction.
- [ ] Machine-readable VM test result.
- [ ] Baseline comparison recorded.

---
