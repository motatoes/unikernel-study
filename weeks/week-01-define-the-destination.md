> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 0 — How to Use This Handbook](../chapters/00-orientation.md)

---

# Week 01 — Define the destination

**Objective:** Decide what you are building and prevent early scope drift.

### Reading

- [ ] CS:APP Chapter 1, *A Tour of Computer Systems*.
- [ ] OSDev: [Introduction](https://wiki.osdev.org/Introduction).
- [ ] OSDev: [Getting Started](https://wiki.osdev.org/Getting_Started).
- [ ] OSDev: [Required Knowledge](https://wiki.osdev.org/Required_Knowledge).
- [ ] OSDev: [Beginner Mistakes](https://wiki.osdev.org/Beginner_Mistakes).

### Implementation

- [ ] Create the repository structure in [What you will build](#3-what-you-will-build).
- [ ] Write `VISION.md` describing the final useful application.
- [ ] Write `NON_GOALS.md` using the scope list later in this document.
- [ ] Write `docs/architecture-decisions/ADR-0001-initial-scope.md`.
- [ ] Draw the final target stack from application to VMM.
- [ ] Choose a baseline comparison guest, such as a minimal NixOS or Alpine VM.

### Verification

- [ ] You can explain why the project is a library OS/unikernel rather than merely a small kernel.
- [ ] You can identify the intended security boundary.
- [ ] You have selected one first application: HTTP service, DNS service, or key-value service.
- [ ] You have committed the vision and non-goals.

### Exit artifact

```text
docs/VISION.md
docs/NON_GOALS.md
docs/architecture-decisions/ADR-0001-initial-scope.md
```

---
