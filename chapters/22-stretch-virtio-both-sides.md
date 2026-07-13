# Stretch Chapter 22 — Implement Both Sides of Virtio

## Goal

Implementing only a guest driver can leave device behavior mysterious; implementing only a host device can leave guest ownership assumptions implicit. This track builds each device on both sides and tests the pair against independent implementations.

## Device order

1. Toy queue device
2. Entropy device
3. Console device
4. Block device
5. Network device
6. Vsock device
7. Optional PCI transport and MSI-X

## Per-device method

For each device:

1. Read the generic virtio facilities and device-specific specification.
2. Write a state machine and feature table.
3. Define queue layouts and buffer ownership.
4. Implement a fake model in hosted tests.
5. Implement the guest driver.
6. Implement the `oc-vmm` device.
7. Test guest driver against QEMU/Firecracker where available.
8. Test an existing guest driver against `oc-vmm` where practical.
9. Fuzz register access and descriptor chains.
10. Benchmark notifications, copies, and exits.

## Cross-implementation matrix

| Guest | Device model | Purpose |
|---|---|---|
| `oc-uk` driver | hosted fake | deterministic unit/state tests |
| `oc-uk` driver | `oc-vmm` | end-to-end ownership and interrupt tests |
| `oc-uk` driver | QEMU | compatibility reference |
| `oc-uk` driver | Firecracker | production-target compatibility |
| Linux driver | `oc-vmm` | independent validation of host device semantics |

Linux-driver interoperability may require a more complete machine/boot environment; it is a stretch within the stretch track.

## Advanced queue topics

After split queues work:

- event-index notification suppression;
- indirect descriptors;
- packed virtqueues;
- multiqueue devices;
- queue reset;
- MSI-X vectors;
- zero-copy or reduced-copy backends;
- vhost/vhost-user comparisons.

Implement one feature at a time and negotiate it only after the entire path supports it.

## Host memory safety

The device model must not create a host slice until it has:

```text
validated GPA + length without overflow
located exactly one registered memory range
checked read/write direction
bounded total chain length
ensured mapping lifetime for the operation
```

Do not cache raw host pointers across memory-layout changes or lifecycle transitions without a strict contract.

## Backend semantics

### Block

Define short reads/writes, flush, read-only mode, capacity changes, cancellation, and backend failure. A guest request may span multiple descriptors and backend operations.

### Network

Define TAP readiness, packet truncation, offload features, multiqueue, rate limiting, and behavior when the guest has no RX buffers.

### Vsock

Define connection table limits, credit accounting, host endpoint authorization, reset, and clone/restore identity.

## Adversarial tests

- descriptor chains that loop;
- chains longer than queue size;
- GPA at end of address space plus overflowing length;
- writable flag opposite to device requirement;
- duplicate completion;
- stale completion after reset;
- register accesses with unexpected width;
- interrupt status read/ack races;
- backend completion after device reset;
- snapshot with requests in each lifecycle state.

## Acceptance criteria

- One generic queue implementation is shared by all guest drivers.
- One defensive guest-memory translation layer is shared by all host devices.
- Every negotiated feature has a conformance test.
- QEMU/Firecracker compatibility tests pass for supported devices.
- Malicious input remains bounded and cannot access host memory outside registered guest regions.
- Snapshot tests prove completion and interrupt state are not duplicated or lost.
