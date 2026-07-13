# Chapter 19 — Targeting Firecracker

## Purpose

Firecracker is a production-oriented KVM VMM with a deliberately constrained machine model. Targeting it forces `oc-uk` to separate platform assumptions from library-OS logic and provides a realistic integration point for Opencomputer.

Consult the current [Firecracker repository and documentation](https://github.com/firecracker-microvm/firecracker) because supported devices and operational guidance evolve.

## Learning objectives

You should be able to:

- package and boot an x86-64 unikernel image through Firecracker's kernel-loading path;
- configure memory, vCPUs, network, block, vsock, entropy, logs, and metrics;
- adapt boot information and virtio transport discovery;
- use vsock as a control channel;
- understand Firecracker snapshot constraints;
- launch through a jailer/isolated worker design;
- compare Firecracker with QEMU and `oc-vmm`.

## Firecracker's design point

Firecracker uses KVM but avoids the broad hardware model of a general machine emulator. It exposes a smaller set of devices and an API-oriented lifecycle suitable for microVM workloads. That reduced model improves startup, attack-surface control, and snapshot tractability, at the cost of general OS/hardware compatibility.

This aligns with a unikernel: the guest can target a known virtual platform rather than probing arbitrary legacy hardware.

## Integration layers

```text
Opencomputer worker
    ↓ API / process supervision
Firecracker + jailer
    ↓ KVM and virtio devices
oc-uk platform backend
    ↓ normalized BootInfo/Device traits
library OS and application
```

Keep Firecracker-specific boot and device-discovery code in one platform module. The network stack and application should be unchanged from QEMU.

## Kernel image and boot ABI

Firecracker's x86-64 kernel-loading support accepts a Linux-style kernel path and, in supported configurations, uncompressed ELF kernel images. Verify the current documented requirements rather than assuming any ELF layout is accepted.

Your image pipeline should produce:

```text
oc-uk.elf
manifest.json or embedded ELF note
Firecracker machine configuration
optional read-only image/root block file
optional writable data block file
```

Validate image load ranges against configured guest memory before invoking Firecracker.

## Device transport

Historically Firecracker's minimal platform emphasized virtio-MMIO; newer capabilities may add or preview other transport/device options. Build `oc-uk` so the virtio device-specific layer is independent of transport. For the first stable target, use the simplest transport supported by the exact Firecracker release you pin.

## Network configuration

Typical path:

```text
guest virtio-net
    ↓
Firecracker TAP backend
    ↓
host bridge/routing/proxy
```

Opencomputer should create and authorize the TAP interface, configure rate limits/policy, then pass the exact resource to the isolated Firecracker process. The guest receives IP configuration through static config, DHCP-like service, or the control channel according to your platform design.

## Block devices

Attach immutable image or data files as declared read-only/read-write devices. Maintain stable device IDs across snapshot and restore. The guest should not infer durable identity solely from enumeration order.

Separate:

```text
boot/application image
persistent data volume
ephemeral scratch
```

Firecracker block snapshots and the underlying data-volume snapshots must be coordinated by Opencomputer.

## Vsock control channel

Vsock is a strong fit for:

- readiness;
- launch configuration;
- secrets;
- logs/metrics control;
- quiesce/checkpoint commands;
- graceful shutdown;
- health probes.

Define an application-independent protocol with:

```text
framing
maximum message size
request IDs
version negotiation
authentication/session binding
timeouts
idempotency
lifecycle state
error codes
```

Do not expose an unrestricted shell over vsock in production.

## Configuration sequence

A worker flow:

```text
1. Resolve and verify image digest.
2. Materialize exact image and block objects.
3. Allocate instance ID, network identity, and vsock CID.
4. Create TAP/backend resources and cgroup.
5. Start Firecracker in isolation.
6. Configure machine, boot source, drives, network, vsock, logs/metrics.
7. Start microVM.
8. Wait for authenticated guest control handshake.
9. Deliver dynamic config/secrets.
10. Wait for application-ready event.
11. Publish instance as running.
```

Every step needs timeout, rollback, and idempotent cleanup.

## Snapshot and restore

Firecracker snapshots capture microVM runtime state and guest memory, while external block-device state must be coordinated separately. Pin and record:

- Firecracker version/format compatibility;
- CPU template/contract;
- machine configuration;
- device IDs and backing snapshot identities;
- guest image/config digest;
- network recreation policy;
- entropy/time/credential refresh policy.

Restore into a prepared environment; do not assume host TAP names or file paths from the original worker remain valid.

## Jailer and host hardening

Production launch should use Firecracker's isolation mechanisms and an outer worker/broker policy. The worker should:

- create only authorized resources;
- place the process in cgroups/namespaces;
- use a minimal chroot/filesystem view;
- drop privileges;
- configure seccomp as supported;
- bound log/metrics output;
- monitor process and vCPU health;
- clean up all resources after exit.

## Compatibility test suite

Run the same guest tests under:

```text
QEMU/KVM
oc-vmm
Firecracker pinned release
```

Cover:

- boot and memory map;
- virtio feature negotiation;
- network TX/RX;
- block read/write/flush;
- vsock request/response;
- entropy;
- clean shutdown;
- snapshot/restore;
- malformed control messages;
- resource limits.

A target-specific failure should not require modifying application logic.

## Debugging playbook

### Image boots under QEMU but not Firecracker

Compare boot protocol, supported ELF/kernel layout, command line, expected load address, CPU features, and device discovery. QEMU's broad compatibility may hide an assumption unsupported by Firecracker.

### Guest network device initializes but no traffic flows

Check TAP ownership/state, host routes, Firecracker interface config, MAC/IP agreement, rate limiter, and guest queue/interrupt handling.

### Vsock connection fails after restore

Check CID uniqueness, host Unix-socket/backend recreation, guest connection state, and whether the protocol is designed to reconnect rather than preserve an external socket.

### Snapshot restore fails on another worker

Check Firecracker version, CPU model/template, device topology/IDs, memory size, kernel/image digest, block identities, and unsupported host-specific state.

## Exercises

1. Boot the same HTTP ELF under QEMU and Firecracker.
2. Implement an authenticated versioned vsock readiness protocol.
3. Attach read-only image, writable data, and ephemeral scratch devices.
4. Snapshot a quiesced instance and restore it on a fresh worker configuration.
5. Compare boot and VM-exit/device metrics with QEMU and `oc-vmm`.
6. Run Firecracker through a worker-created jail/cgroup with only preauthorized resources.

## Review questions

1. Which Firecracker-specific details should remain outside the library OS core?
2. Why must device-specific virtio logic be transport independent?
3. What belongs on vsock versus the tenant network?
4. Which snapshot objects are external to VMM memory state?
5. Why should restore recreate host network resources?
6. What extra isolation exists outside the KVM boundary?

## Opencomputer connection

Firecracker can be Opencomputer's production microVM backend while `oc-vmm` remains a reference/learning implementation and QEMU remains the broad debug backend. The Opencomputer runtime interface should hide VMM-specific API details behind one lifecycle contract but preserve VMM/version/device metadata for compatibility and incident analysis.
