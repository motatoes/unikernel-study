> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 15 — Library Operating Systems and Unikernel Architecture](../chapters/15-library-os-unikernel-design.md)
- [Chapter 16 — Application SDKs, C ABI, and Limited POSIX Compatibility](../chapters/16-application-abi-posix.md)

---

# Week 53 — Project structure, boot ABI, and platform abstraction

**Objective:** Create the final library-OS repository and normalized platform contract.

### Reading

- [ ] Revisit Solo5 manifest and ABI material.
- [ ] Revisit MirageOS composition material.
- [ ] Review ELF note format.

### Implementation

Create this structure:

```text
oc-uk/
├── arch/x86_64/
├── kernel/boot/
├── kernel/memory/
├── kernel/interrupt/
├── kernel/time/
├── kernel/executor/
├── kernel/entropy/
├── platform/hosted/
├── platform/qemu/
├── platform/firecracker/
├── drivers/virtio/
├── libos/io/
├── libos/net/
├── libos/fs/
├── libos/sync/
├── libos/task/
├── libc-shim/
├── sdk/
├── apps/
├── tests/
├── fuzz/
└── benches/
```

- [ ] Define `BootInfo`.
- [ ] Define a `Platform` trait.
- [ ] Define normalized device descriptors.
- [ ] Embed a versioned manifest as an ELF note.
- [ ] Implement a hosted Linux backend for unit/integration tests.
- [ ] Define ABI compatibility rules.

Suggested platform interface:

```rust
pub trait Platform {
    fn memory_regions(&self) -> &[MemoryRegion];
    fn monotonic_time(&self) -> core::time::Duration;
    fn fill_entropy(&self, output: &mut [u8]) -> Result<(), PlatformError>;
    fn devices(&self) -> &[DeviceDescriptor];
    fn shutdown(&self, reason: ShutdownReason) -> !;
}
```

### Verification

- [ ] Core library-OS tests run as ordinary Linux processes.
- [ ] Manifest parser rejects incompatible versions.
- [ ] QEMU-specific types do not leak into application-facing APIs.
- [ ] ABI documents define ownership, layout, and lifetime.

### Exit artifact

```text
oc-uk/
docs/BOOT-ABI.md
docs/PLATFORM-ABI.md
```

---
