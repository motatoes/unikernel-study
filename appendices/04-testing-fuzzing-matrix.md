# Appendix D — Testing, Fuzzing, and Failure-Injection Matrix

## Test pyramid

```text
many fast hosted unit/property tests
        ↓
parser and state-machine fuzzing
        ↓
small VM subsystem tests
        ↓
full end-to-end tests
        ↓
fewer controlled performance/chaos tests
```

## Subsystem matrix

| Subsystem | Unit/property | VM integration | Fuzz | Failure injection | Benchmark |
|---|---|---|---|---|---|
| Address/layout | overflow, canonicality, alignment | invalid mapping fault | structured values | OOM/invalid map | page map cost |
| ELF/manifest | parser and plan | boot exact image | bytes/sections | truncation/overlap | parse/load time |
| Allocator | random allocate/free | exhaustion | operation sequence | OOM/double-free | alloc latency/fragmentation |
| Trap/IDT | frame layout | deliberate vectors | limited state | nested/double fault | handler cost |
| Executor | state transitions | task/timer workload | event sequence | lost wake/cancel | poll latency/fairness |
| Virtqueue | chains/index wrap | real device | descriptors/used ring | reset/stale completion | notifications/operation |
| Network parser | protocol corpus | TAP traffic | packet bytes | loss/reorder/exhaustion | pps/throughput/latency |
| Block/log | record recovery | virtio-block | metadata bytes | crash after writes | IOPS/flush/recovery |
| KVM loader | memory plan | boot under oc-vmm | image/config | unsupported CPU/exit | startup/exits |
| Device model | guest-memory bounds | guest driver | MMIO/descriptors | backend stall/reset | exits/copies |
| Control protocol | message state | vsock | frames/state sequence | timeout/replay | request latency |
| Snapshot | section decoder | restore | bytes/sections | wrong block/CPU/version | size/create/restore |

## Property examples

### ELF loader

- Validation never mutates guest memory.
- A successful plan's ranges are nonoverlapping and in bounds.
- Zero-fill exactly covers `memsz - filesz`.
- Entry lies in one executable range.

### Frame allocator

- Allocated frames are unique.
- Free plus allocated plus reserved equals total tracked frames.
- Freeing an allocated frame permits later reuse.
- Invalid frees never corrupt metadata.

### Virtqueue

- Every submitted head is completed or invalidated by reset exactly once.
- Free plus allocated descriptor count equals queue size.
- No chain contains more descriptors than queue size.
- Index wrap does not change logical operation ordering.

### Executor

- A task occupies one lifecycle state.
- Completion cannot be followed by wake/poll.
- Cancellation releases wait registrations.
- Timer deadlines are processed in nondecreasing order.

## VM test design

A VM test should:

1. boot a minimal feature set;
2. run one deterministic scenario;
3. produce structured pass/fail details;
4. shut down explicitly;
5. have a host timeout;
6. retain serial/VMM logs on failure;
7. identify the exact image/configuration.

Avoid one giant integration image that makes failures ambiguous.

## Fuzz harness rules

- No network or filesystem dependency unless required.
- Strict memory/time limits.
- Parser API returns errors rather than panics.
- Corpus includes valid examples and hand-built edge cases.
- Crashes are minimized and promoted to regression tests.
- Coverage is monitored, but semantic assertions matter more than raw percentage.
- Security-sensitive findings are triaged privately.

## Failure-injection catalog

### CPU/runtime

```text
out of memory
stack overflow
unexpected vector
long interrupt disable
stuck task
panic while lock held
```

### Device

```text
reset in flight
malformed completion
lost interrupt
interrupt storm
backend timeout
short read/write
read-only violation
```

### Network

```text
loss
duplication
reordering
corruption
MTU mismatch
ARP exhaustion
connection flood
slow client
```

### Storage

```text
crash after every write
flush failure
capacity mismatch
wrong snapshot parent
long snapshot chain
partial object upload
```

### Platform

```text
worker restart during create
VMM crash
control-plane retry
artifact cache corruption
snapshot incompatibility
network resource recreation failure
secret service timeout
```

## Continuous integration tiers

### Per commit

```text
format/lint
hosted unit/property tests
short fuzz smoke corpus
ELF permission/manifest checks
small QEMU boot tests
```

### Nightly

```text
QEMU + Firecracker compatibility matrix
long randomized allocator/queue tests
network loss/reordering tests
snapshot restore matrix
selected fuzz campaigns
```

### Controlled benchmark environment

```text
boot distributions
memory/RSS
network/block throughput and p99
VM exits
snapshot size/time/restore
baseline comparison
```

Performance tests should not block ordinary commits based on noisy microbenchmarks. Use broad smoke thresholds in CI and analyze precise regressions in controlled workers.
