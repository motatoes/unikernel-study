> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 13 — Building a Small KVM VMM](../chapters/13-kvm-vmm.md)

---

# Week 41 — Create and run a KVM VM

**Objective:** Use the raw KVM API to execute a tiny guest.

### Reading

- [ ] KVM API: general description.
- [ ] KVM, VM, and vCPU file descriptors.
- [ ] `KVM_GET_API_VERSION`, `KVM_CREATE_VM`, memory-region APIs, `KVM_CREATE_VCPU`, and `KVM_RUN`.

### Implementation

- [ ] Open `/dev/kvm`.
- [ ] Verify API version and required capabilities.
- [ ] Create a VM.
- [ ] Allocate and register guest memory.
- [ ] Create a vCPU.
- [ ] Map `kvm_run`.
- [ ] Run a tiny real-mode guest that performs port I/O and halts.
- [ ] Handle I/O, HLT, shutdown, and unexpected exits.

### Verification

- [ ] Guest output is visible through an I/O exit.
- [ ] Every exit reason is logged.
- [ ] Unexpected exits produce enough state to debug.

### Exit artifact

```text
oc-vmm/src/kvm.rs
oc-vmm/examples/real-mode-hello.rs
```

---
