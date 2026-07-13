> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 14 — VMM Snapshots, Isolation, and Defensive Device Models](../chapters/14-vmm-snapshot-security.md)

---

# Week 45 — Stopped-state snapshots

**Objective:** Save and restore a quiesced VM with explicit compatibility metadata.

### Reading

- [ ] KVM get/set registers and events.
- [ ] Firecracker snapshot documentation.

### Implementation

Define a snapshot containing:

```text
format version
image digest
configuration digest
guest memory
vCPU general registers
vCPU special registers
interrupt state
device state
clock state
```

- [ ] Implement stop-the-world capture.
- [ ] Implement restore into a new VMM process.
- [ ] Validate image and configuration identity.
- [ ] Reject unsupported snapshot versions.

### Verification

- [ ] Restored guest continues from the expected point.
- [ ] Timers do not jump backward.
- [ ] Wrong image digest is rejected.
- [ ] Device state matches pre-snapshot state.

### Exit artifact

```text
oc-vmm/src/snapshot/
docs/SNAPSHOT-FORMAT-draft.md
```

---
