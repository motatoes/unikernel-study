# Stretch Chapter 24 — SDK, Observability, Fuzzing, and the Full Opencomputer Runtime

## Goal

Turn the core technical system into a developer platform. This track focuses on application experience, reusable templates, compatibility reporting, operational observability, continuous fuzzing, and distributed runtime services.

## SDK workstream

Build:

```text
oc-uk-sdk
├── native Rust APIs
├── procedural entry macro
├── C headers and bindings
├── application templates
├── hosted test runtime
├── local QEMU/Firecracker runner
├── build/size report
├── compatibility checker
└── documentation generator
```

### Developer experience requirements

A developer should be able to:

```bash
oc init --template http-rust my-service
cd my-service
oc test
oc run --target hosted
oc run --target qemu
oc build --target firecracker
oc inspect result/image
```

The same configuration schema should work locally and in Opencomputer. Secrets must use local development providers and never be copied into build artifacts.

## Application templates

Provide examples for:

- HTTP service;
- UDP/DNS-like service;
- persistent key-value service;
- scheduled/background task;
- C application using the stable ABI;
- service with a read-only embedded asset archive.

Every template should include tests, metrics, graceful shutdown, and checkpoint hooks.

## Compatibility service

For source-ported applications, build a report showing:

```text
resolved symbols
implemented interfaces
unsupported calls
semantic deviations
selected libc/runtime
threading model
filesystem assumptions
image size contribution
```

Make this report an artifact of every build rather than tribal knowledge.

## Observability workstream

Add:

- OpenTelemetry-compatible host spans where appropriate;
- stable event schemas;
- guest-to-host metrics snapshots;
- crash support bundles;
- fleet dashboards for boot, failure, memory, exits, and restore;
- per-image regression views;
- trace sampling and data-retention policy.

Do not forward unlimited tenant-provided labels or logs into the platform metrics system.

## Continuous fuzzing workstream

Maintain separate campaigns for:

```text
ELF and manifest parsers
network protocols
virtqueue guest driver
virtio host device models
vsock/control protocol
block metadata
snapshot envelope and sections
SDK configuration parser
```

Requirements:

- sanitizer and debug configurations where supported;
- corpus stored and deduplicated;
- minimized regressions checked into tests;
- coverage reporting;
- resource/time limits;
- private handling for security-sensitive findings.

## Distributed runtime workstream

Services:

```text
API/control plane
build service
artifact registry/cache
scheduler
worker agent
network controller/proxy
volume and snapshot service
identity/secrets service
metrics/log/trace pipeline
compatibility catalog
```

### End-to-end durability

Use a transactional control-plane model so an instance or snapshot is visible only after required objects and metadata commit. Workers reconcile desired state against observed local state after restart.

### Migration strategy

Start with stop-and-restore:

```text
quiesce
data checkpoint
runtime snapshot
transfer/cache objects
prepare destination
restore
health validation
switch endpoint
retire source
```

Live migration is a later research extension requiring iterative dirty-page transfer, device/backend coordination, connection handling, and strict CPU/VMM compatibility.

### Multi-tenant policy

Define:

- per-tenant build and runtime quotas;
- image-signing and admission policy;
- allowed network egress;
- volume/snapshot ownership;
- secret scopes;
- debug feature policy;
- snapshot retention and deletion;
- audit events.

## Advanced timeline

| Weeks | Focus |
|---:|---|
| 83–84 | SDK command-line and hosted/QEMU runner |
| 85–86 | C ABI packaging and compatibility reports |
| 87–88 | Observability schemas, support bundles, dashboards |
| 89–90 | Continuous fuzzing infrastructure |
| 91–94 | Distributed build/registry/scheduler/worker integration |
| 95–96 | Cross-worker checkpoint/restore and failure exercises |

## Final acceptance criteria

- New applications can be created, tested, run, and built through one SDK.
- Build artifacts include compatibility, size, test, provenance, and security reports.
- Every critical parser and device state machine has continuous fuzz coverage.
- Opencomputer can schedule by immutable digest and compatibility contract.
- Checkpoint/restore works across workers with authenticated objects and recreated networking.
- Operations dashboards identify image-specific regressions and failure classes.
- Tenant quotas, secret handling, audit, and cleanup are documented and tested.
