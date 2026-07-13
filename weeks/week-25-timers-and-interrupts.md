> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 7 — Interrupts, Time, Entropy, and Lifecycle State](../chapters/07-interrupts-time-entropy.md)

---

# Week 25 — Timers and interrupts

**Objective:** Add an interrupt-driven source of time.

### Reading

- [ ] *Writing an OS in Rust*: Hardware Interrupts.
- [ ] Intel SDM APIC material.
- [ ] OSDev: [APIC](https://wiki.osdev.org/APIC).
- [ ] OSDev: [APIC Timer](https://wiki.osdev.org/APIC_Timer).
- [ ] OSDev: [PIT](https://wiki.osdev.org/Programmable_Interval_Timer).
- [ ] OSDev: [HPET](https://wiki.osdev.org/HPET).

### Implementation

- [ ] Select and document the initial interrupt controller/timer path.
- [ ] Program a periodic or deadline timer.
- [ ] Acknowledge interrupts correctly.
- [ ] Maintain monotonic ticks.
- [ ] Keep the interrupt handler minimal and defer work.

### Verification

- [ ] Tick rate is within measured tolerance.
- [ ] Interrupt count remains stable under load.
- [ ] No interrupt storm after acknowledgement.
- [ ] Timer tests pass under several QEMU CPU settings.

### Exit artifact

```text
oc-kernel/src/time/
oc-kernel/src/arch/x86_64/apic.rs
```

---
