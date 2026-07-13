> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 6 — Boot, Build, Test, and Debug Bare-Metal Code](../chapters/06-boot-build-debug.md)

---

# Week 22 — Kernel testing and diagnostics

**Objective:** Make failures reproducible before adding complexity.

### Reading

- [ ] *Writing an OS in Rust*: Testing.
- [ ] OSDev: [Testing](https://wiki.osdev.org/Testing).
- [ ] OSDev: [Serial Ports](https://wiki.osdev.org/Serial_Ports).
- [ ] OSDev: [Debugging](https://wiki.osdev.org/Debugging).

### Implementation

- [ ] Add a kernel test runner.
- [ ] Add a panic handler with source location where available.
- [ ] Add a QEMU exit device or equivalent machine-readable status.
- [ ] Add GDB command files and symbol loading.
- [ ] Add structured boot milestones.
- [ ] Run kernel tests in CI.

### Verification

- [ ] A passing test exits with success.
- [ ] A deliberate panic exits with failure and emits diagnostics.
- [ ] CI detects timeout and missing output.

### Exit artifact

```text
oc-kernel/tests/
scripts/kernel-test
.gdbinit-oc-kernel
```

---
