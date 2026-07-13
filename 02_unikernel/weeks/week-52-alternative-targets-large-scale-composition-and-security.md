> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 15 — Library Operating Systems and Unikernel Architecture](../chapters/15-library-os-unikernel-design.md)
- [Chapter 17 — Security Engineering for a Unikernel Platform](../chapters/17-unikernel-security.md)

---

# Week 52 — Alternative targets, large-scale composition, and security

**Objective:** Finalize the architecture of `oc-uk` before implementation.

### Reading

- [ ] *Unikernels as Processes*.
- [ ] *Programming Unikernels in the Large*.
- [ ] *A Security Perspective on Unikernels*.
- [ ] *Assessing Unikernel Security*.

### Practical work

- [ ] Complete the comparison matrix below.
- [ ] Write the `oc-uk` architecture decision record.
- [ ] Draft the threat model.
- [ ] Define hosted Linux, QEMU, `oc-vmm`, and Firecracker targets.
- [ ] Define native API and limited POSIX strategy.

### Required comparison matrix

| Dimension | MirageOS | Solo5 | Unikraft | `oc-uk` |
|---|---|---|---|---|
| Application languages | | | | |
| Application ABI | | | | |
| Host/platform ABI | | | | |
| Build specialization | | | | |
| Scheduling | | | | |
| Memory management | | | | |
| Networking | | | | |
| Block storage | | | | |
| libc/POSIX support | | | | |
| VM/process targets | | | | |
| Configuration | | | | |
| Debugging | | | | |
| Snapshot model | | | | |
| Isolation boundary | | | | |

### Verification

The architecture decision record justifies:

- [ ] one address space;
- [ ] static application linkage;
- [ ] native API first;
- [ ] limited POSIX later;
- [ ] cooperative execution first;
- [ ] virtio devices;
- [ ] platform-independent driver interfaces;
- [ ] hosted, QEMU, `oc-vmm`, and Firecracker backends;
- [ ] narrow host-control ABI;
- [ ] content-addressed ELF artifact.

### Exit artifact

```text
docs/architecture-decisions/ADR-0006-oc-uk-architecture.md
docs/THREAT-MODEL-draft.md
docs/unikernel-comparison.md
```

## Phase 6 gate

- [ ] Reviewed the foundational library-OS and unikernel papers.
- [ ] Built at least one MirageOS and one Unikraft application.
- [ ] Studied Solo5’s ABI/substrate split.
- [ ] Completed architecture and threat-model drafts.
- [ ] Can defend `oc-uk`’s application and platform interfaces.

---
