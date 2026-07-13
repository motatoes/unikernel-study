> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 7 — Interrupts, Time, Entropy, and Lifecycle State](../chapters/07-interrupts-time-entropy.md)
- [Chapter 6 — Boot, Build, Test, and Debug Bare-Metal Code](../chapters/06-boot-build-debug.md)

---

# Week 30 — Time, entropy, logs, and embedded files

**Objective:** Complete the minimum runtime services needed by higher layers.

### Reading

- [ ] OSDev: [Time and Date](https://wiki.osdev.org/Time_And_Date).
- [ ] OSDev: [Random Number Generator](https://wiki.osdev.org/Random_Number_Generator).
- [ ] OSDev: [Stack Trace](https://wiki.osdev.org/Stack_Trace).

### Implementation

- [ ] Provide monotonic time in a stable unit.
- [ ] Define wall-clock time as externally supplied configuration.
- [ ] Implement entropy initialization and an RNG interface.
- [ ] Implement structured logging with level and subsystem.
- [ ] Add symbolized or partially symbolized backtraces where practical.
- [ ] Embed and inspect a read-only archive.

### Verification

- [ ] Monotonic time never moves backward in normal execution.
- [ ] Entropy failure has an explicit policy.
- [ ] Logs include image/build identity.
- [ ] Embedded files are reproducibly included in the image.

### Exit artifact

```text
oc-kernel/src/time/
oc-kernel/src/entropy/
oc-kernel/src/logging/
oc-kernel/src/archive/
```

## Phase 3 gate

- [ ] Boots through Limine under QEMU.
- [ ] Correct GDT, TSS, IDT, and fault reporting.
- [ ] Timer interrupts and monotonic time.
- [ ] Owned page tables and W^X.
- [ ] Frame, heap, stack, and DMA allocation.
- [ ] Cooperative tasks and wake-ups.
- [ ] Structured logging and automated tests.

---
