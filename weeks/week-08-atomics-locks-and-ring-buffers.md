> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 4 — Concurrency, Scheduling, and Event-Driven Execution](../chapters/04-concurrency-execution.md)

---

# Week 08 — Atomics, locks, and ring buffers

**Objective:** Build the synchronization primitives later required by interrupts and virtqueues.

### Reading

- [ ] CS:APP Chapter 6 review for cache behaviour.
- [ ] OSTEP Chapter 28, *Locks*.
- [ ] Rustonomicon: atomics and `Send`/`Sync`.
- [ ] Linux kernel memory-barrier documentation.
- [ ] OSDev: [Spinlock](https://wiki.osdev.org/Spinlock).

### Implementation

- [ ] Implement a spin lock.
- [ ] Define an interrupt-safe locking strategy for future kernel use.
- [ ] Implement a bounded single-producer/single-consumer ring.
- [ ] Create a deliberately broken weak-ordering version.
- [ ] Add stress tests and model-based tests where practical.
- [ ] Document false sharing and cache-line alignment concerns.

### Verification

- [ ] No lost, duplicated, or reordered entries under stress.
- [ ] Acquire/release points are justified in comments.
- [ ] Lock tests include contention and recursive-acquisition failure.
- [ ] You can explain why a virtqueue requires publication ordering.

### Exit artifact

```text
labs/lockfree-ring/
labs/spinlock/
docs/notes/week-08-memory-ordering.md
```

## Phase 1 gate

- [ ] Freestanding ELF with custom entry point.
- [ ] Custom linker script and linker map.
- [ ] ELF inspector with malformed-input tests.
- [ ] Correct assembly/Rust ABI crossing.
- [ ] Typed address and MMIO primitives.
- [ ] Working ring buffer with documented memory ordering.

---
