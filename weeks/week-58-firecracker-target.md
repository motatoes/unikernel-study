> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 19 — Targeting Firecracker](../chapters/19-firecracker-target.md)

---

# Week 58 — Firecracker target

**Objective:** Boot the same application image under Firecracker.

### Reading

- [ ] Firecracker [design](https://github.com/firecracker-microvm/firecracker/blob/main/docs/design.md).
- [ ] Firecracker [kernel/rootfs setup](https://github.com/firecracker-microvm/firecracker/blob/main/docs/rootfs-and-kernel-setup.md).
- [ ] Firecracker [vsock documentation](https://github.com/firecracker-microvm/firecracker/blob/main/docs/vsock.md).

### Implementation

- [ ] Implement the Firecracker-compatible boot path.
- [ ] Normalize memory map and command line into `BootInfo`.
- [ ] Discover/configure virtio-MMIO devices.
- [ ] Validate virtio-net.
- [ ] Validate virtio-block.
- [ ] Validate vsock.
- [ ] Implement Firecracker-compatible shutdown signalling.
- [ ] Add a Firecracker integration-test harness.

### Verification

- [ ] Same HTTP application works under QEMU and Firecracker.
- [ ] Same data format works on the attached block device.
- [ ] Readiness and shutdown work through vsock.
- [ ] Platform-specific code is confined to the Firecracker backend.

### Exit artifact

```text
oc-uk/platform/firecracker/
tests/firecracker/
```

---
