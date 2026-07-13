# Chapter 11 — Block Storage, Persistence, and Checkpoint Semantics

## Purpose

A bootable image, mutable application data, a container-style writable layer, and a VM memory snapshot are different objects with different correctness properties. This chapter develops the storage model needed for a durable Opencomputer workload.

## Learning objectives

You should be able to:

- implement and validate virtio-block requests;
- distinguish read completion from durable write completion;
- explain cache, flush, and ordering semantics;
- choose an appropriate first on-disk format;
- define crash-consistent application checkpoints;
- separate immutable image, data volume, and VM runtime snapshot;
- design snapshot identity and restore validation.

## Storage layers

```text
application data model
    ↓
filesystem or custom format
    ↓
block interface
    ↓
virtio-block driver
    ↓
VMM file-backed/block-backed device
    ↓
host filesystem or block service
    ↓
physical storage
```

Each layer may buffer or reorder. A successful write submission is not necessarily durable. Define what `flush` means at every boundary you control.

## Virtio-block request model

A block request typically contains:

```text
request header
    type and sector

data buffers
    readable by device for writes
    writable by device for reads

status byte
    written by device
```

Validate:

- sector arithmetic and overflow;
- buffer length alignment where required;
- negotiated capacity;
- request type and features;
- status byte;
- completion ID;
- read-only device policy.

Do not report success to the application until the device status is checked.

## First persistence format

Do not begin by implementing ext4. Choose one of:

1. **Embedded read-only archive** for application assets.
2. **Read-only simple filesystem** for image contents.
3. **Append-only key-value log** for a persistent capstone.
4. **SQLite with a custom VFS** once the block and flush model is sound.

An append-only log teaches durability with limited surface area. A record format might contain:

```text
magic
format version
record length
sequence number
operation
key length/value length
payload
checksum
commit marker or atomic framing
```

Recovery scans until the first invalid or incomplete record and rebuilds an in-memory index.

## Crash consistency

A system can crash between any two durable writes. For every update, identify the invariants that must survive every prefix of the write sequence.

Use fault injection:

```text
perform operation
simulate crash after write 0
recover and verify
simulate crash after write 1
recover and verify
...
```

Power-loss behavior is not identical to a process crash; host caches and storage devices complicate the real durability model. For the educational system, clearly define the guarantees supplied by the VMM-backed block device and `flush` path.

## Storage object model for Opencomputer

Recommended separation:

```text
unikernel image
  immutable, content-addressed ELF and manifest

data volume
  durable application state, independently snapshotted

runtime scratch
  ephemeral caches and temporary files

VM runtime snapshot
  guest memory, vCPU, and device state for fast continuation
```

The durable application identity should not depend solely on a VM memory snapshot. Memory snapshots are tightly coupled to CPU/VMM/device versions and can preserve undesirable transient state.

## Quiescing

A storage-consistent checkpoint requires coordination:

```text
1. Stop admission of new application work.
2. Wait for or cancel in-flight operations.
3. Commit application-level transactions.
4. Submit block flush and wait for completion.
5. Freeze mutations to the volume.
6. Snapshot the block device.
7. Optionally snapshot guest memory/device state.
8. Resume or stop.
```

Filesystem freeze alone does not guarantee application-level consistency. A database may have its own transaction and cache protocol.

## Snapshot identity

A snapshot manifest should include:

```json
{
  "format": 1,
  "image_digest": "sha256:...",
  "config_digest": "sha256:...",
  "vmm": { "kind": "firecracker", "version": "..." },
  "cpu_contract": "...",
  "memory_object": "...",
  "vmm_state_object": "...",
  "block_snapshots": [
    { "device_id": "data0", "snapshot_id": "...", "writable": true }
  ],
  "network_restore_policy": "recreate",
  "created_at": "...",
  "integrity": "..."
}
```

Do not trust paths embedded in an untrusted snapshot. Resolve object IDs through the control plane and validate all lengths and versions before allocation.

## Snapshot chains

Copy-on-write block snapshots are efficient, but deep chains degrade performance and complicate retention. Define:

- maximum chain depth;
- flatten/consolidation policy;
- parent reference counting;
- garbage collection;
- encryption key lifecycle;
- cross-worker transfer format.

Content-addressed immutable layers can be shared; mutable data snapshots need explicit ownership and retention.

## Restore protocol

Before guest execution:

1. authenticate snapshot manifest;
2. verify image and configuration digests;
3. verify VMM and device compatibility;
4. validate CPU contract;
5. attach exact block snapshots to exact device IDs;
6. allocate guest memory safely;
7. decode state with bounded parsers;
8. restore vCPU/device state;
9. establish control channel;
10. reseed entropy and apply time policy;
11. recreate platform network state;
12. release the guest to run.

## Debugging playbook

### Data disappears after reported successful write

Check whether the application waited for completion, whether the status byte indicated success, whether a flush was issued, and whether host backing storage honors the expected durability contract.

### Snapshot restores but application state is inconsistent

Check application quiescing, transaction commit, in-flight block requests, data-volume snapshot timing, and mismatch between memory state and block snapshot.

### Restore works only on original worker

Look for host paths, CPU-specific state, local cache dependencies, TAP names, vsock CIDs, and VMM-version coupling not represented in metadata.

### Snapshot chain performance degrades

Measure read amplification and chain depth; consolidate rather than repeatedly stacking overlays indefinitely.

## Exercises

1. Implement an append-only key-value log with crash-after-every-write tests.
2. Add virtio-block read, write, flush, and read-only policy tests.
3. Produce a consistent block snapshot without a memory snapshot, then restore application state.
4. Produce a memory snapshot paired with the wrong block snapshot and ensure admission rejects it.
5. Clone one data snapshot into two writable children and verify isolation.
6. Fuzz the snapshot manifest and VMM-state decoder.

## Review questions

1. Why is write completion not always durability?
2. Which layer defines application consistency?
3. Why should a memory snapshot not be the canonical data format?
4. What must be frozen before a block snapshot is taken?
5. Why are deep copy-on-write chains operationally dangerous?
6. Which restore checks must occur before allocating large buffers?

## Opencomputer connection

Opencomputer should expose three separate operations: **checkpoint data**, **suspend runtime**, and **create template**. Checkpointing snapshots durable volumes after application quiesce. Suspending additionally captures VM state for fast continuation. Creating a template produces a new immutable application artifact. Keeping these semantics separate avoids users accidentally treating fragile VM state as durable data.
