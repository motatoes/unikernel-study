> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 2 — The x86-64 Machine Model](../chapters/02-x86-64-machine-model.md)

---

# Week 07 — Exceptions and trap-frame design

**Objective:** Understand asynchronous and exceptional control transfer before handling it on bare metal.

### Reading

- [ ] CS:APP Chapter 8, *Exceptional Control Flow*.
- [ ] Intel SDM topics: exception classes, error codes, interrupt gates, and privilege transitions.
- [ ] OSDev: [Exceptions](https://wiki.osdev.org/Exceptions).
- [ ] OSDev: [Interrupt Service Routines](https://wiki.osdev.org/Interrupt_Service_Routines).
- [ ] OSDev: [Interrupt Descriptor Table](https://wiki.osdev.org/Interrupt_Descriptor_Table).

### Implementation

- [ ] Build a host-side trap-frame simulator.
- [ ] Model exceptions with and without hardware error codes.
- [ ] Model nested exceptions and a double-fault condition.
- [ ] Define a stable register-dump format.
- [ ] Write a dispatcher that separates architecture entry from handler logic.

### Verification

- [ ] Tests cover at least divide error, invalid opcode, general protection, and page fault.
- [ ] The dispatcher cannot confuse vectors with and without error codes.
- [ ] You can draw the stack layout for same-privilege and privilege-changing entries.

### Exit artifact

```text
labs/trap-frame-simulator/
docs/notes/week-07-exceptions.md
```

---
