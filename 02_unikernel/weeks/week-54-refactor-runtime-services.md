> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 3 — Physical Memory, Virtual Memory, and Allocation](../chapters/03-memory-management.md)
- [Chapter 4 — Concurrency, Scheduling, and Event-Driven Execution](../chapters/04-concurrency-execution.md)
- [Chapter 7 — Interrupts, Time, Entropy, and Lifecycle State](../chapters/07-interrupts-time-entropy.md)

---

# Week 54 — Refactor runtime services

**Objective:** Move proven kernel mechanisms into clean, testable library-OS components.

### Reading

- [ ] Revisit your allocator, executor, time, entropy, and logging notes.
- [ ] Review unsafe Rust and data-layout rules for all public low-level APIs.

### Implementation

- [ ] Refactor physical-memory and mapping code.
- [ ] Refactor heap, stack, and DMA allocation.
- [ ] Refactor executor, wait queues, and timers.
- [ ] Refactor entropy and RNG.
- [ ] Refactor logging and panic diagnostics.
- [ ] Define initialization ordering.
- [ ] Make read-only configuration immutable after initialization.

### Verification

- [ ] Hosted tests cover most platform-independent logic.
- [ ] Bare-metal QEMU tests cover architecture-specific paths.
- [ ] Every unsafe module contains invariants and safety contracts.
- [ ] W^X and guard-page tests continue to pass.

### Exit artifact

```text
oc-uk/kernel/
oc-uk/tests/runtime/
```

---
