> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 3 — Physical Memory, Virtual Memory, and Allocation](../chapters/03-memory-management.md)

---

# Week 27 — Own the mappings and enforce W^X

**Objective:** Create, change, and remove mappings safely.

### Reading

- [ ] *Writing an OS in Rust*: Paging Implementation.
- [ ] Intel SDM page permissions, NX, TLB invalidation, and canonical-address rules.

### Implementation

- [ ] Implement `map`, `unmap`, and permission-update operations.
- [ ] Unmap page zero.
- [ ] Make code RX, read-only data R, and writable data RW+NX.
- [ ] Add a temporary mapping mechanism.
- [ ] Add TLB invalidation where required.
- [ ] Reject overlapping or invalid mappings.

### Verification

- [ ] Write to code triggers a fault.
- [ ] Execute from heap triggers a fault.
- [ ] Access to page zero triggers a fault.
- [ ] Mapping tests include huge-page boundary cases where relevant.

### Exit artifact

```text
oc-kernel/src/memory/paging.rs
oc-kernel/tests/page-permissions.rs
```

---
