# Chapter 14 — VMM Snapshots, Isolation, and Defensive Device Models

## Purpose

A VMM handles untrusted guest-controlled state while holding access to host memory, files, network interfaces, and KVM. Snapshot decoders additionally process large, privileged, persistent inputs. This chapter treats VMM isolation and snapshot design as primary architecture concerns.

## Learning objectives

You should be able to:

- define a complete stopped-state snapshot;
- coordinate vCPU, device, memory, timer, and block state;
- version and validate snapshot formats;
- identify guest-to-host attack surfaces;
- sandbox a VMM with least privilege and narrow host resources;
- design defensive virtio device models;
- distinguish warm resume from portable application checkpointing.

## Snapshot components

A stopped VM snapshot may contain:

```text
manifest and format version
VM configuration and feature policy
vCPU general/special/extended state
interrupt-controller state
device model state
guest memory or memory-page index
clock/timer state
external resource identities
integrity/authentication metadata
```

Writable block storage should generally be snapshot independently and referenced by identity. Embedding huge mutable disk images inside the VMM-state format makes lifecycle management and consistency harder.

## Stop-the-world protocol

A correct baseline is a stopped snapshot:

```text
request guest quiesce
    ↓
guest drains and confirms quiesced
    ↓
pause/kick all vCPUs
    ↓
wait for vCPU barrier
    ↓
drain device event handlers
    ↓
freeze/flush external block state
    ↓
serialize devices and CPU state
    ↓
capture guest memory
    ↓
commit authenticated manifest
```

Do not attempt live migration until this path is reliable and heavily tested.

## State ordering

Define dependencies. For example, a used virtqueue entry visible in memory must correspond to device state that will not re-complete the same request after restore. A pending interrupt must be consistent with both device status and virtual interrupt-controller state.

The snapshot point is one logical instant across:

```text
vCPUs
memory
devices
interrupts
timers
external storage
```

Stopping only the vCPU while device threads continue mutating memory does not produce a coherent snapshot.

## Versioning

Use an envelope that supports:

- magic and format version;
- bounded section table;
- per-section type/version/length/checksum;
- optional compression identified explicitly;
- endianness and architecture;
- image/configuration digest;
- required VMM capability set.

Avoid serializing raw Rust structs. Compiler layout, enum representation, and dependency versions are not durable file-format contracts.

Use explicit integer widths and checked decoding.

## Restore validation

Validation must happen before large allocation or host-resource attachment when possible:

1. authenticate envelope;
2. check size limits;
3. parse with checked arithmetic;
4. verify architecture and format versions;
5. verify image/configuration identity;
6. check CPU/device compatibility;
7. resolve storage/network references through authorized control-plane objects;
8. allocate guest memory;
9. restore bounded state;
10. establish lifecycle and control channels;
11. release vCPUs.

A snapshot is equivalent to code and memory supplied to a privileged component. Treat it as hostile unless authenticated and authorized.

## Guest-to-host attack surfaces

A malicious guest controls or influences:

- virtio descriptor addresses, lengths, flags, and chains;
- device register access size and order;
- packet contents;
- block request patterns;
- vsock protocol messages;
- shutdown/reboot behavior;
- interrupt pressure;
- memory contents captured into snapshots;
- timing and resource consumption.

Device models must validate every guest-provided offset before translating it to host memory.

## Defensive virtio device implementation

For each descriptor chain:

```text
limit total descriptors
reject loops
check every GPA range
check read/write direction
bound total byte length
copy or pin according to clear ownership rules
handle partial backend operations
complete once
never trust guest-provided identifiers as host indexes
```

Rate-limit expensive paths and cap in-flight requests. The host must remain responsive even if the guest never consumes completions.

## Process isolation

A production VMM should run with:

- dedicated unprivileged identity;
- no unnecessary capabilities;
- restricted filesystem view;
- pre-opened or brokered resources;
- cgroup CPU/memory/I/O limits;
- namespaces where appropriate;
- seccomp or equivalent syscall filtering;
- no ambient network access except declared backends;
- bounded logging and core-dump policy.

Firecracker's jailer and thread-specific filters illustrate defense in depth: KVM is the primary hardware boundary, but the userspace VMM should still be treated as attackable.

## Resource broker pattern

Avoid giving the VMM permission to open arbitrary host files or create arbitrary TAP devices. A privileged worker/broker can:

1. authorize a workload specification;
2. create/open exact resources;
3. pass file descriptors to the VMM;
4. drop or isolate privileges;
5. monitor lifecycle externally.

This narrows the impact of a compromised device model.

## Time and entropy after restore

After restore:

- rebase monotonic time according to policy;
- update wall-clock anchor;
- reseed guest randomness;
- rotate ephemeral platform credentials;
- allocate a fresh vsock identity if cloning;
- recreate network interfaces/routes;
- invalidate external connection assumptions.

Two clones of one snapshot must not share identity accidentally.

## Warm snapshot versus durable checkpoint

A warm snapshot optimizes continuation of an exact runtime. It is coupled to VMM, CPU contract, device topology, and transient state.

A durable checkpoint stores application-defined state and durable volumes in a versioned format that survives runtime upgrades. Opencomputer should support both and make the distinction explicit.

## Debugging playbook

### Restored VM repeats a completion

Compare queue memory, device in-flight state, used ring, interrupt state, and backend request state at the snapshot point. The device was likely not drained or serialized atomically.

### Snapshot parser crashes before authentication

Reorder the envelope design. Authentication and coarse size limits should be checked before complex decoding; parsing itself must still be memory safe and bounded.

### VMM memory grows under malicious guest

Check unbounded in-flight requests, log amplification, descriptor-chain allocation, packet queues, backend retries, and snapshot buffering.

### Clone has duplicate network identity

Separate image identity, instance identity, MAC/IP assignment, vsock CID, and cryptographic identity. Regenerate instance-scoped values after clone.

## Exercises

1. Design a sectioned snapshot format and write a fuzzable decoder.
2. Snapshot after every device-request lifecycle state and verify restore behavior.
3. Run a malicious descriptor fuzzer against the host device model.
4. Launch the VMM inside a restricted namespace/cgroup/seccomp profile.
5. Pass TAP and block file descriptors from a broker rather than opening paths in the VMM.
6. Clone one snapshot twice and test identity, entropy, storage, and networking separation.

## Review questions

1. Why is pausing the vCPU alone insufficient for a coherent snapshot?
2. Why should block storage be independently identified?
3. Why is serializing raw Rust structs unsafe as a durable format?
4. Which inputs can a malicious guest control in a virtio device model?
5. What resources should be pre-opened by a broker?
6. How does a warm snapshot differ from an application checkpoint?

## Opencomputer connection

Opencomputer should own snapshot authentication, encryption, retention, compatibility policy, and object references. The VMM should serialize only runtime state; the control plane should assemble the complete checkpoint object and authorize restore. This separation keeps tenant metadata and storage policy out of the VMM while preventing snapshots from becoming unauthenticated host-local files.
