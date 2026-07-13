# Appendix G — Source Archaeology Guide

Reading production systems is essential, but opening a large repository without a question is inefficient. Use a hypothesis-driven method.

## Method

For one mechanism:

1. State the question.
2. Find the public interface or configuration entry.
3. Locate the central type and state machine.
4. Trace one success path.
5. Trace one failure/reset path.
6. Identify synchronization and ownership.
7. Find tests for the path.
8. Record what is architecture versus implementation accident.

Write a one-page trace containing:

```text
Question:
Entry point:
Central data structures:
Call/data path:
Ownership transitions:
Locks/threads:
Failure/reset path:
Tests:
Design lesson for oc-uk/oc-vmm:
```

## Linux paths to inspect

Repository: <https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/>

### x86 entry and traps

Search for:

```text
x86 exception entry stubs
common interrupt entry
page-fault handler
IDT setup
TSS/IST setup
```

Goal: understand frame normalization and separation of assembly entry from C logic—not to copy the complexity.

### Memory

Inspect:

```text
x86 page-table types/macros
TLB invalidation interfaces
physical page allocation overview
vmalloc/direct map concepts
```

Goal: identify how production code encodes page-table levels, permissions, and architecture variation.

### KVM UAPI and tests

Inspect:

```text
include/uapi/linux/kvm.h
Documentation/virt/kvm/api.rst
tools/testing/selftests/kvm
```

Goal: connect ioctl documentation to actual structures and failure tests.

### Virtio ring helpers

Inspect the generic ring implementation and one simple driver. Goal: compare descriptor lifecycle and ordering with your implementation.

## QEMU paths to inspect

Repository: <https://gitlab.com/qemu-project/qemu>

Start with:

- a tiny MMIO device;
- memory-region registration;
- one virtio device's queue processing;
- TAP networking backend;
- kernel/ELF loading;
- migration state descriptions.

Questions:

```text
How are MMIO ranges registered?
How are guest addresses translated?
How is device state versioned?
How are interrupts raised/lowered?
What prevents malformed guest descriptors from escaping memory ranges?
```

QEMU's generality creates complexity. Extract principles, not structure.

## Firecracker paths to inspect

Repository: <https://github.com/firecracker-microvm/firecracker>

Trace:

```text
API request → VM configuration
VM start → KVM/vCPU creation
network API → TAP/backend/device
block API → backend/device
vsock configuration → host Unix socket
snapshot request → pause/state/memory handling
metrics emission
jailer setup
```

Record which responsibilities are in Firecracker, jailer, external launcher, and guest.

## rust-vmm paths to inspect

Organization: <https://github.com/rust-vmm>

Look for current locations of:

```text
KVM bindings/ioctl wrappers
guest-memory abstractions
Linux/ELF/kernel loaders
eventfd/epoll utilities
virtio queue/device abstractions
rate limiting
```

Repositories may be reorganized; pin a current monorepo/crate version and avoid old examples that point to archived locations.

## Solo5

Repository: <https://github.com/Solo5/solo5>

Trace:

```text
application manifest
binding ABI
HVT/tender startup
network/block interfaces
shutdown
ABI versioning
```

Question: what is the smallest host interface on which a library OS can rely?

## MirageOS

Organization: <https://github.com/mirage>

Trace one HTTP application from configuration to target-specific build. Focus on:

```text
explicit module dependencies
time/entropy/network signatures
hosted versus hypervisor target
configuration graph
library selection
```

Question: how can application code remain independent of one operating-system backend?

## Unikraft

Organization: <https://github.com/unikraft>

Trace:

```text
configuration selection
library dependency resolution
allocator/scheduler selection
libc and syscall shim
application port
network driver/stack integration
```

Question: where does modularity create power, and where does it create configuration/compatibility complexity?

## smoltcp

Repository: <https://github.com/smoltcp-rs/smoltcp>

Trace:

```text
device receive/transmit token
interface poll
socket-set dispatch
next poll deadline
TCP connection state
```

Question: what contract must `oc-uk` provide so the stack remains event driven without busy polling?

## Review discipline

Never paste production code into your project until you can state:

- its license and attribution requirements;
- its hidden dependencies;
- the invariant it enforces;
- which platform assumptions it carries;
- how you will test it independently.
