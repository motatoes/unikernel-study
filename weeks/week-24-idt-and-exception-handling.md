> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 2 — The x86-64 Machine Model](../chapters/02-x86-64-machine-model.md)

---

# Week 24 — IDT and exception handling

**Objective:** Catch, classify, and report architectural faults.

### Reading

- [ ] *Writing an OS in Rust*: CPU Exceptions.
- [ ] *Writing an OS in Rust*: Double Faults.
- [ ] Intel SDM exception and IDT sections.
- [ ] OSDev: [Interrupt Descriptor Table](https://wiki.osdev.org/Interrupt_Descriptor_Table).
- [ ] OSDev: [Exceptions](https://wiki.osdev.org/Exceptions).

### Implementation

- [ ] Install an IDT.
- [ ] Handle divide error, invalid opcode, general protection, page fault, and double fault.
- [ ] Use the dedicated IST stack for double fault.
- [ ] Print registers, vector, error code, instruction pointer, and fault address.
- [ ] Add deliberate tests for every handler.

### Verification

- [ ] Each deliberate fault reaches the correct handler.
- [ ] A page fault prints the faulting virtual address and decoded error bits.
- [ ] Double-fault handling does not triple fault.

### Exit artifact

```text
oc-kernel/src/arch/x86_64/idt.rs
oc-kernel/tests/exceptions.rs
```

---
