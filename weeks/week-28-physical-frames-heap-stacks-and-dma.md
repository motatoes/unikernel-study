> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 3 — Physical Memory, Virtual Memory, and Allocation](../chapters/03-memory-management.md)

---

# Week 28 — Physical frames, heap, stacks, and DMA

**Objective:** Build allocation layers with explicit ownership and telemetry.

### Reading

- [ ] OSTEP Chapter 17 review.
- [ ] *Writing an OS in Rust*: Heap Allocation.
- [ ] *Writing an OS in Rust*: Allocator Designs.
- [ ] OSDev: [Page Frame Allocation](https://wiki.osdev.org/Page_Frame_Allocation).
- [ ] OSDev: [Heap](https://wiki.osdev.org/Heap).

### Implementation

- [ ] Implement an early bump allocator.
- [ ] Implement a physical-frame bitmap or buddy allocator.
- [ ] Implement a heap allocator.
- [ ] Implement stack allocation with guard pages.
- [ ] Implement a page-aligned, physically suitable DMA allocator.
- [ ] Add allocation counters and leak diagnostics.

### Verification

- [ ] Exhaustion returns a controlled error or panic policy.
- [ ] Double-free and invalid-free tests are detected.
- [ ] Guard-page overflow triggers a page fault.
- [ ] DMA buffers satisfy alignment and contiguity requirements you declare.

### Exit artifact

```text
oc-kernel/src/memory/frame_allocator.rs
oc-kernel/src/memory/heap.rs
oc-kernel/src/memory/dma.rs
```

---
