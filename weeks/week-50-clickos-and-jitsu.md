> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 15 — Library Operating Systems and Unikernel Architecture](../chapters/15-library-os-unikernel-design.md)

---

# Week 50 — ClickOS and Jitsu

**Objective:** Study network-specialized images and request-driven activation.

### Reading

- [ ] *ClickOS and the Art of Network Function Virtualization*.
- [ ] *Jitsu: Just-In-Time Summoning of Unikernels*.

### Practical work

- [ ] Map the boot path and network attachment model in both systems.
- [ ] Identify control-plane responsibilities.
- [ ] Design an Opencomputer request-to-instance sequence.
- [ ] Define readiness and termination events.
- [ ] Estimate which startup milestones dominate latency in your current kernel.

### Verification

- [ ] Sequence diagram covers request, image selection, VM creation, network setup, readiness, and teardown.
- [ ] You can distinguish cold boot, restored boot, and already-running routing.
- [ ] You have identified metrics needed for startup optimization.

### Exit artifact

```text
docs/paper-reviews/clickos.md
docs/paper-reviews/jitsu.md
docs/opencomputer-startup-sequence.md
```

---
