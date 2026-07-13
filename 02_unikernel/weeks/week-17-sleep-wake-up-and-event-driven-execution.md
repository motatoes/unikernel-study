> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 4 — Concurrency, Scheduling, and Event-Driven Execution](../chapters/04-concurrency-execution.md)
- [Chapter 5 — OS Interfaces Through xv6](../chapters/05-os-interfaces-xv6.md)

---

# Week 17 — Sleep, wake-up, and event-driven execution

**Objective:** Understand blocking, wake-ups, and the event-driven model that will shape the unikernel.

### Reading

- [ ] OSTEP Chapters 31–33.
- [ ] xv6 Chapter 9, *Sleep and wake-up*.
- [ ] OSDev: [Context Switching](https://wiki.osdev.org/Context_Switching).

### Implementation

- [ ] Trace xv6 sleep and wake-up paths.
- [ ] Build a Linux-hosted event reactor using `epoll`, timers, and explicit wake-ups.
- [ ] Add cancellation and deadline handling.
- [ ] Create a lost-wake-up test and document how correct ordering prevents it.

### Verification

- [ ] The reactor handles timer and I/O events without busy waiting.
- [ ] Lost-wake-up test fails in the broken version and passes in the correct version.
- [ ] You can compare threads, blocking, callbacks, and async tasks.

### Exit artifact

```text
labs/event-reactor/
docs/notes/week-17-events.md
```

---
