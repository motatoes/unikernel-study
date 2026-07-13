> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 4 — Concurrency, Scheduling, and Event-Driven Execution](../chapters/04-concurrency-execution.md)

---

# Week 29 — Cooperative execution

**Objective:** Run multiple tasks without process semantics.

### Reading

- [ ] *Writing an OS in Rust*: Async/Await.
- [ ] OSTEP Chapter 33 review.
- [ ] OSDev: [Context Switching](https://wiki.osdev.org/Context_Switching).
- [ ] OSDev: [Scheduling Algorithms](https://wiki.osdev.org/Scheduling_Algorithms).

### Implementation

Choose either stackful cooperative contexts or a `Future`-based executor as the primary model. Document the choice.

- [ ] Implement task creation.
- [ ] Implement a ready queue.
- [ ] Implement wake-up.
- [ ] Implement a sleep/deadline queue.
- [ ] Implement cancellation.
- [ ] Ensure interrupt handlers only enqueue work.

### Verification

- [ ] Multiple tasks make progress.
- [ ] Sleeping tasks do not busy wait.
- [ ] Wake-up cannot be lost.
- [ ] Cancellation releases resources.

### Exit artifact

```text
oc-kernel/src/executor/
docs/architecture-decisions/ADR-0003-execution-model.md
```

---
