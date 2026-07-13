> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 4 — Concurrency, Scheduling, and Event-Driven Execution](../chapters/04-concurrency-execution.md)
- [Chapter 5 — OS Interfaces Through xv6](../chapters/05-os-interfaces-xv6.md)

---

# Week 15 — Scheduling policy

**Objective:** Understand what a scheduler decides and what it does not.

### Reading

- [ ] OSTEP Chapters 7–9.
- [ ] xv6 Chapter 8, *Scheduling*.
- [ ] OSDev: [Scheduling Algorithms](https://wiki.osdev.org/Scheduling_Algorithms).

### Implementation

- [ ] Add scheduler telemetry.
- [ ] Implement a small priority, lottery, or weighted scheduler modification.
- [ ] Measure fairness and starvation behaviour.
- [ ] Create CPU-bound and I/O-bound test workloads.

### Verification

- [ ] Scheduler metrics reveal expected policy behaviour.
- [ ] You can explain context-switch cost and scheduling latency.
- [ ] You can state why the first unikernel starts with cooperative execution.

### Exit artifact

```text
xv6-labs/week-15/
docs/notes/week-15-scheduling.md
```

---
