> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 15 — Library Operating Systems and Unikernel Architecture](../chapters/15-library-os-unikernel-design.md)

---

# Week 49 — Solo5

**Objective:** Study a narrow execution substrate between library OS and host.

### Reading

- [ ] Solo5 repository documentation and architecture material.
- [ ] Solo5 manifest, ABI, and tender/binding documentation.

### Practical work

- [ ] Draw the split among application, library OS, bindings, and tender.
- [ ] Inspect ABI versioning and application manifest design.
- [ ] Run a hardware-virtualized and process-style target where practical.
- [ ] Compare its host interface with your platform abstraction.

### Verification

- [ ] You can explain why a narrow substrate improves portability.
- [ ] You can identify which responsibilities belong in `oc-uk` and which belong in a VMM/tender.
- [ ] You have proposed an embedded manifest format for `oc-uk`.

### Exit artifact

```text
docs/paper-reviews/solo5.md
docs/architecture-decisions/ADR-0005-image-manifest.md
```

---
