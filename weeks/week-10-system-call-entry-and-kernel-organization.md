> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 5 — OS Interfaces Through xv6](../chapters/05-os-interfaces-xv6.md)

---

# Week 10 — System-call entry and kernel organization

**Objective:** Trace a controlled transition from unprivileged code into a kernel.

### Reading

- [ ] OSTEP Chapter 6, *Limited Direct Execution*.
- [ ] xv6 Chapter 2.
- [ ] xv6 Sections 4.3–4.4 on system-call handling.
- [ ] OSDev: [Context Switching](https://wiki.osdev.org/Context_Switching).

### Implementation

- [ ] Complete the xv6 system-call lab.
- [ ] Add syscall tracing.
- [ ] Add system-call accounting by process and call number.
- [ ] Add invalid-pointer and invalid-length tests.
- [ ] Trace user registers through trampoline, trap frame, dispatch, and return.

### Verification

- [ ] Invalid userspace pointers fail safely.
- [ ] Traces show argument and return-value flow.
- [ ] You can explain why a single-address-space unikernel does not need the same syscall boundary.

### Exit artifact

```text
xv6-labs/week-10/
docs/notes/week-10-syscall-path.md
```

---
