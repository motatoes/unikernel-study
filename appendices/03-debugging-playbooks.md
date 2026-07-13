# Appendix C — Debugging Playbooks

## The first rule

Reduce the failing system until the symptom still occurs. Disable unrelated devices, tasks, allocators, optimizations, and concurrency. Preserve the exact image and command line.

## Boot failure checklist

```text
[ ] Correct image loaded
[ ] ELF architecture/class valid
[ ] Entry lies in executable mapping
[ ] Stack mapped, writable, canonical, aligned
[ ] .bss initialized
[ ] Boot information address valid
[ ] CPU mode/control registers expected
[ ] GDT/IDT/TSS bases and limits valid
[ ] Serial route and UART initialized
[ ] QEMU automatic reboot disabled
[ ] GDB attached to exact symbols
```

## Page fault checklist

Capture:

```text
RIP
CR2
error code
CR3
RSP and stack bounds
page-table walk at each level
mapping owner and permissions
current lifecycle/task
```

Then ask:

- Is the address canonical?
- Was the access read, write, execute, user, or reserved-bit related?
- Is the failing instruction itself mapped?
- Is this a guard-page success rather than an unexpected fault?
- Did the allocator return a frame still in use?
- Did a page-table update require invalidation?

## Triple fault checklist

- Run without reboot.
- Inspect last reset state.
- Break before loading IDT/GDT/CR3.
- Trigger one known exception at a time.
- Verify exception stubs with and without error codes.
- Verify double-fault IST stack.
- Confirm handler does not allocate or take unsafe locks.
- Confirm return frame and instruction are correct.

## Allocator corruption checklist

```text
[ ] Run allocator tests hosted
[ ] Enable poison on alloc/free
[ ] Add red zones/canaries
[ ] Validate free address/domain/alignment
[ ] Detect double-free
[ ] Record allocation owner/call site
[ ] Disable concurrency
[ ] Check DMA/device still owns buffer
[ ] Check integer overflow in layout
```

## Virtio checklist

Initialization:

```text
[ ] Device identity/version correct
[ ] Reset completed
[ ] Status transitions valid
[ ] Features read/written from correct pages
[ ] Only implemented features accepted
[ ] Queue size supported
[ ] Descriptor/avail/used GPAs correct
[ ] Queue marked ready
[ ] Driver ready status set
```

Submission:

```text
[ ] Buffers alive and aligned
[ ] Descriptor direction flags correct
[ ] Chain terminates
[ ] Head published to avail ring
[ ] Memory barrier before avail index/notify
[ ] Correct queue notified
```

Completion:

```text
[ ] Used index acquired correctly
[ ] Completion head is in flight
[ ] Device-written length/status bounded
[ ] Descriptors reclaimed once
[ ] Waiter woken once
[ ] Interrupt acknowledged
```

## Networking checklist

Host side:

```text
[ ] TAP exists and is up
[ ] VMM owns correct fd/interface
[ ] Bridge/routes/namespaces correct
[ ] MAC/IP match configuration
[ ] Egress/firewall policy permits test
```

Guest side:

```text
[ ] RX buffers posted
[ ] TX completion recycling works
[ ] Ethernet EtherType endian correct
[ ] ARP target/source correct
[ ] IPv4 total/header lengths checked
[ ] Checksums correct
[ ] Stack polled on packet and timer events
[ ] Listener bound and executor wakes
```

Use `tcpdump` to identify the last visible boundary.

## KVM/VMM checklist

```text
[ ] KVM API version supported
[ ] Required capabilities present
[ ] Guest memory slot ranges correct
[ ] vCPU mmap size correct
[ ] Registers/sregs initialized coherently
[ ] CPUID contract sufficient
[ ] Exit reason classified
[ ] MMIO width/range validated
[ ] Interrupt state consistent
[ ] vCPU kick/pause works
```

For exit storms, aggregate by reason and guest RIP.

## Snapshot checklist

Before capture:

```text
[ ] Guest quiesced
[ ] New work admission stopped
[ ] Block operations drained/flushed
[ ] All vCPUs paused at barrier
[ ] Device threads drained or paused
[ ] Interrupt/device/queue states coherent
[ ] External block snapshot complete
```

Before restore:

```text
[ ] Manifest authenticated
[ ] Size/version/architecture validated
[ ] Image/config digest matches
[ ] CPU/VMM/device compatibility passes
[ ] Exact block snapshots attached
[ ] Network resources recreated
[ ] Entropy reseed scheduled
[ ] Time policy applied
[ ] Control session reestablished
```

## Release-only failure checklist

Suspect:

```text
undefined behavior
missing volatile access
incorrect aliasing
uninitialized data
stack alignment
integer overflow
race/memory ordering
link-time dead-code elimination
panic/unwind profile differences
```

Compare disassembly and reproduce with sanitizers/hosted tests where possible.

## Evidence bundle

Every unrecoverable failure should preserve a bounded bundle:

```text
image and config digest
source/build version
VMM/version/command line
host CPU/kernel
serial tail
structured guest crash record
VMM stderr and exit reason
selected metrics
snapshot/object IDs when relevant
reproduction command
```
