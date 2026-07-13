# Chapter 18 — Performance, Measurement, and Observability

## Purpose

Unikernels are often justified by boot speed, small memory footprint, and efficient I/O. Those claims are meaningful only when measurement boundaries and baselines are explicit. Observability must be designed into the guest and VMM so performance can be explained rather than merely reported.

## Learning objectives

You should be able to:

- define boot and readiness milestones precisely;
- measure guest and host memory separately;
- classify VM exits and device notifications;
- benchmark network and block paths without misleading warm-cache effects;
- collect structured guest/VMM metrics with bounded overhead;
- profile code size and allocation sources;
- compare Linux, Unikraft, QEMU, Firecracker, and `oc-vmm` fairly;
- turn performance regressions into reproducible tests.

## Measurement principles

1. **Define the boundary.** “Boot time” may mean process start, first guest instruction, kernel ready, network ready, or application ready.
2. **Record the environment.** Host CPU, kernel, VMM, compiler, build profile, memory, vCPUs, and networking matter.
3. **Separate cold and warm paths.** Page cache, image cache, JIT, DNS, ARP, and connection reuse can dominate.
4. **Use distributions.** Report p50/p95/p99 and sample count, not only an average.
5. **Change one variable.** Avoid comparing different VMMs, images, CPUs, and application configurations simultaneously.
6. **Measure overhead of measurement.** Debug logs can alter scheduling and exits.

## Boot timeline

Instrument guest milestones with a monotonic timestamp and host correlation ID:

```text
host create requested
host VMM process started
KVM VM created
guest first instruction
boot info validated
page tables ready
allocator ready
devices ready
network configured
control channel ready
application ready
first request accepted
```

Where possible, correlate host and guest clocks through an initial anchor rather than assuming their timestamps share an epoch.

## Memory metrics

Distinguish:

```text
configured guest RAM
host virtual allocation
host resident set size
private dirty pages
shared/file-backed pages
VMM overhead
KVM/kernel overhead estimate
guest allocated frames
guest heap live/peak
device buffers
stack reservation versus use
snapshot working set
```

A VM configured with 128 MiB does not necessarily consume 128 MiB resident host memory immediately. Conversely, host RSS can include page cache or shared mappings not attributable in a simple way.

## Image-size analysis

Produce a build report by section and component:

```text
.text
.rodata
.data
.bss
unwind/debug metadata
embedded assets
network stack
allocator
crypto
compatibility layer
application
```

Use linker maps, `size`, `nm`, `objdump`, and Rust tooling. Track size over time and set budgets for optional features.

## VM-exit analysis

Classify exits by:

- reason;
- guest RIP or symbol;
- device;
- count;
- total and max handling time;
- request/packet correlation.

Common avoidable sources:

- excessive MMIO notifications;
- periodic timers;
- port polling;
- small I/O batches;
- unnecessary CPUID/MSR exits;
- interrupt storms;
- guest halt/wakeup behavior.

Optimize only after identifying the dominant category.

## Network benchmarks

Measure separately:

```text
raw virtio throughput
TCP throughput
request throughput
latency under controlled concurrency
small and large payloads
packet rate
interrupts/packet
VM exits/request
CPU use in guest/VMM/host
```

Use a stable host network topology and account for NAT or proxies. Compare the same application behavior and request limits.

## Storage benchmarks

Measure:

- sequential and random reads/writes;
- request sizes;
- queue depth;
- flush latency;
- data-volume snapshot creation;
- restore read amplification;
- copy-on-write chain depth;
- cache-cold versus cache-warm behavior.

Do not benchmark an append-only log against a mature filesystem without clarifying semantic differences.

## Structured observability

Guest events should be compact and machine readable:

```json
{
  "ts_mono_ns": 123456,
  "level": "info",
  "component": "virtio-net",
  "event": "rx_batch",
  "instance": "...",
  "image": "sha256:...",
  "fields": { "packets": 32, "bytes": 48120 }
}
```

Avoid arbitrary high-cardinality labels. Metrics should be bounded and reset/overflow behavior defined.

Core guest metrics:

```text
allocated/free frames
heap live/peak/failures
tasks by state
executor poll duration
timer lag
network packets/bytes/drops
connections by state
block operations/errors/latency
virtqueue notifications/completions
control messages
quiesce duration
panic count
```

Core VMM metrics:

```text
VM exits
vCPU run/blocked time
MMIO/I/O operations
interrupt injections
device queue depth
backend errors
memory resident/dirty
snapshot phase duration
lifecycle state duration
```

## Tracing

Use correlation IDs across:

```text
control request
VMM lifecycle event
guest control message
application request
block/network operation
snapshot operation
```

A small fixed-size guest trace ring can preserve recent events for crash reports without streaming every event continuously.

## Benchmark harness

A benchmark entry should record:

```yaml
commit: ...
image_digest: ...
host_cpu: ...
host_kernel: ...
vmm: ...
vmm_version: ...
vcpus: 1
memory_mib: 64
network_mode: tap-bridge
build_profile: release
samples: 1000
warmup: 100
workload: ...
```

Store raw results, not only rendered charts.

## Debugging playbook

### Boot time regresses

Compare milestone deltas. Determine whether time moved into VMM creation, guest initialization, device negotiation, network configuration, or application startup.

### Throughput rises but p99 worsens

Inspect batching, cooperative task starvation, interrupt suppression, queue depth, and oversized executor work units.

### Guest reports low memory but host RSS is high

Inspect VMM buffers, snapshot mappings, page cache, TAP/backend queues, logging, and unreturned guest memory pages that the host cannot reclaim.

### Benchmark is unstable

Pin CPU frequency/governor where appropriate, isolate cores, avoid noisy neighbors, record temperature, increase samples, and separate cold/warm runs.

## Exercises

1. Add all boot milestones and render a timeline.
2. Produce a per-component image-size report.
3. Add VM-exit counters to `oc-vmm` and identify the top three causes.
4. Benchmark polling versus interrupt-driven virtio-net.
5. Compare the same HTTP application under minimal Linux, Unikraft, `oc-uk`/QEMU, and `oc-uk`/Firecracker.
6. Add a CI performance smoke test with broad regression thresholds and a separate controlled benchmark suite.

## Review questions

1. Which milestone should define “application ready”?
2. Why is configured guest RAM not equal to host RSS?
3. How can batching improve throughput but hurt tail latency?
4. Which exit metrics reveal an overly chatty device interface?
5. Why must raw benchmark data be retained?
6. Which metric labels can cause observability-system overload?

## Opencomputer connection

Opencomputer needs performance data at the scheduling boundary: startup distribution by image/worker class, resident memory, restore working set, data-volume latency, and VMM exit/device load. The guest manifest can include expected resource minima, but placement should be driven by measured behavior. Observability should link build digest, instance, snapshot, and worker without exposing tenant secrets.
