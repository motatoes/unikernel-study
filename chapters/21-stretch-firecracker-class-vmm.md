# Stretch Chapter 21 — From Educational VMM to Firecracker-Class MicroVM Monitor

## Goal

Extend `oc-vmm` from a teaching VMM into a small, hardened microVM monitor with a stable control API, multiple vCPUs, production virtio devices, snapshots, metrics, and process isolation.

This remains a learning/research implementation. “Firecracker-class” describes the architecture and discipline, not a claim of equivalent maturity, audit depth, or production history.

## Advanced architecture

```text
oc-vmm process
├── API/control thread
├── lifecycle state machine
├── resource broker interface
├── VM memory manager
├── vCPU threads
├── interrupt controller
├── epoll/event manager
├── virtio-MMIO/PCI bus
│   ├── net
│   ├── block
│   ├── vsock
│   ├── rng
│   └── console
├── rate limiters
├── snapshot manager
├── metrics/logging
└── sandbox setup
```

## 12-week implementation sequence

### Weeks 61–62 — Control plane and lifecycle

- Define an OpenAPI or gRPC API.
- Implement `Created → Configured → Running → Paused → Stopped` transitions.
- Reject configuration mutation after boot unless explicitly supported.
- Add idempotent stop/delete and structured errors.
- Create integration tests for every invalid transition.

### Weeks 63–64 — Multi-vCPU

- Start application processors through the guest's selected boot protocol.
- Add one thread per vCPU.
- Implement pause barriers and vCPU kicks.
- Define CPU affinity and scheduling policy.
- Serialize all vCPU state coherently.
- Add tests for halted and interrupt-waiting vCPUs.

### Weeks 65–66 — Interrupt architecture

- Implement a virtual interrupt controller appropriate to your machine model.
- Add eventfd/irqfd where supported and understood.
- Route device interrupts to selected vCPUs.
- Add level and edge semantics tests.
- Measure injection latency and exit reduction.

### Weeks 67–68 — Production devices

- Harden virtio-net, block, vsock, and entropy.
- Add queue-size limits and negotiated feature policies.
- Add rate limiting for bytes and operations.
- Support read-only and writable block backends.
- Add backend error and cancellation semantics.

### Week 69 — Snapshot format and dirty tracking

- Add versioned section encoding.
- Track dirty guest pages where practical.
- Support full and incremental memory capture experiments.
- Bind state to CPU/device/image configuration.
- Fuzz restore decoding.

### Week 70 — Metrics and tracing

- Export lifecycle, vCPU, exit, queue, backend, memory, and snapshot metrics.
- Add correlation IDs and bounded event rings.
- Create a support bundle with configuration and recent events.

### Week 71 — Sandboxing

- Launch with dedicated UID/GID.
- Restrict filesystem and network namespace.
- Apply cgroups and syscall filtering.
- Accept pre-opened resources from a broker.
- Verify the VMM cannot open arbitrary host paths.

### Week 72 — Adversarial qualification

- Fuzz device models and API parser.
- Run malicious guest queue patterns.
- Test memory pressure, backend stalls, interrupt storms, and API races.
- Compare behavior with Firecracker and crosvm where useful.
- Write a security and performance report.

## Additional engineering topics

### Device backend isolation

For stronger isolation, consider moving high-risk or complex backends into separate processes or vhost-user-style services. Evaluate the trade-off between additional IPC complexity and containment.

### CPU templates

Define named virtual CPU profiles rather than exposing raw host features. A profile should be testable on all intended worker classes and versioned for snapshot compatibility.

### Live update versus replacement

Prefer immutable VMM versions and workload replacement. In-process live update is a major research project. If runtime snapshots cross versions, create an explicit compatibility matrix and conversion strategy.

### Confidential-computing extension

After the base VMM is robust, study SEV-SNP and TDX. These change attestation, memory visibility, device trust, snapshots, and debugging. Treat them as a separate track rather than adding scattered flags to the normal design.

## Acceptance criteria

- Stable documented control API.
- Multi-vCPU guest boots and shuts down correctly.
- Virtio-net, block, vsock, and entropy pass guest/host fuzz tests.
- Snapshot restore is versioned, authenticated by the platform, and compatibility checked.
- VMM runs with no arbitrary path or network access.
- Host remains bounded under a malicious guest.
- Performance and exit profiles are reproducible.
- Opencomputer can swap Firecracker and `oc-vmm` behind one runtime interface.
