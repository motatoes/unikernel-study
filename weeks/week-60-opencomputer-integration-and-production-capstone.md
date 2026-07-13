> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 20 — Integrating `oc-uk` With Opencomputer](../chapters/20-opencomputer-runtime.md)
- [Chapter 18 — Performance, Measurement, and Observability](../chapters/18-performance-observability.md)

---

# Week 60 — Opencomputer integration and production capstone

**Objective:** Turn the research implementation into a reproducible runtime artifact and operational workflow.

### Reading

- [ ] Review your own architecture, ABI, snapshot, and threat-model documents.
- [ ] Review Nix closure and reproducible-build techniques required by the build pipeline.

### Implementation

Build this pipeline:

```text
application source
        │
        ▼
Nix-pinned compiler and dependencies
        │
        ▼
ukbuild component selection
        │
        ▼
static link
        │
        ▼
ELF + embedded manifest
        │
        ▼
validation and signing
        │
        ▼
content-addressed artifact store
```

- [ ] Implement `ukbuild` or an equivalent build wrapper.
- [ ] Validate architecture, ABI version, device requirements, and ELF permissions.
- [ ] Calculate image and configuration digests.
- [ ] Produce a signed artifact manifest.
- [ ] Implement an Opencomputer runtime adapter.
- [ ] Allocate TAP interfaces and vsock CIDs.
- [ ] Attach block devices and snapshot identities.
- [ ] Capture logs, readiness, metrics, and termination reason.
- [ ] Run the production capstone under load and failure injection.
- [ ] Complete the benchmark matrix.

Suggested artifact manifest:

```json
{
  "media_type": "application/vnd.opencomputer.unikernel.v1",
  "digest": "sha256:...",
  "architecture": "x86_64",
  "platforms": ["qemu-kvm", "firecracker"],
  "minimum_memory_bytes": 67108864,
  "vcpus": 1,
  "devices": ["net", "vsock", "block"],
  "application_abi": 1,
  "platform_abi": 1,
  "config_digest": "sha256:...",
  "signature": "..."
}
```

Suggested runtime interface:

```go
type UnikernelSpec struct {
    ImageDigest  string
    Platform     string
    VCPUs        int
    MemoryBytes  uint64
    KernelArgs   []string
    Network      NetworkSpec
    BlockDevices []BlockSpec
    VsockCID     uint32
    Secrets      []SecretRef
}

type UnikernelRuntime interface {
    Create(ctx context.Context, spec UnikernelSpec) (Instance, error)
    Start(ctx context.Context, id string) error
    Stop(ctx context.Context, id string) error
    Snapshot(ctx context.Context, id string) (Snapshot, error)
    Restore(ctx context.Context, snapshot Snapshot) (Instance, error)
    Delete(ctx context.Context, id string) error
}
```

### Verification

- [ ] Clean build is reproducible from a fresh checkout.
- [ ] Artifact identity is content-addressed and signed.
- [ ] QEMU and Firecracker runtimes launch the same logical application.
- [ ] Block snapshot and VM runtime snapshot remain distinct objects.
- [ ] Cold start, restore, load, shutdown, and fault tests pass.
- [ ] Documentation is sufficient for another engineer to build and operate the system.

### Exit artifact

```text
tools/ukbuild/
opencomputer/runtime/unikernel/
docs/ARCHITECTURE.md
docs/OPERATIONS.md
docs/PERFORMANCE.md
docs/PORTING.md
```

## Phase 7 gate

- [ ] One Rust application and one C application.
- [ ] QEMU, `oc-vmm`, and Firecracker targets.
- [ ] Virtio-net, block, and vsock.
- [ ] Stable application and platform ABI documents.
- [ ] Threat model and fuzzing.
- [ ] Quiesced snapshot/restore path.
- [ ] Nix-pinned reproducible artifact pipeline.
- [ ] Opencomputer runtime adapter.
- [ ] Final benchmark and operations documentation.

---
