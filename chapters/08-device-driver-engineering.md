# Chapter 8 — Device Driver Engineering: MMIO, DMA, Queues, and Interrupts

## Purpose

Virtual devices are simpler than arbitrary physical hardware, but they still require precise contracts. Driver bugs often come from address-domain confusion, descriptor lifetime mistakes, missing ordering, incomplete reset handling, or assumptions about well-behaved devices.

## Learning objectives

You should be able to:

- distinguish port I/O, MMIO, and DMA;
- design typed register blocks and safe access wrappers;
- define device state machines and reset behavior;
- implement producer/consumer queues with ownership transfer;
- separate interrupt handling from deferred completion work;
- validate device-provided lengths, indexes, and identifiers;
- test a driver against a fake device before booting a VM.

## Three communication mechanisms

### Port I/O

The CPU executes special instructions to access an I/O port namespace. This is common for legacy x86 devices such as a serial port.

### MMIO

Registers are mapped into the address space. Accesses must use volatile semantics and correct widths. The compiler must not remove, combine, or reorder accesses in ways prohibited by the device contract.

### DMA

The device reads or writes memory without the CPU copying every byte. The driver gives the device a device-visible address and must keep the memory valid until ownership returns.

## Register access design

Avoid exposing arbitrary pointers. Wrap registers by width and access mode:

```rust
pub struct ReadOnly<T> { /* ... */ }
pub struct WriteOnly<T> { /* ... */ }
pub struct ReadWrite<T> { /* ... */ }
```

A register contract should record:

- offset;
- width;
- permitted access sizes;
- reset value;
- reserved bits;
- write-one-to-clear behavior;
- side effects;
- ordering requirements.

Never use a packed Rust struct and take references to potentially unaligned fields. Compute offsets or use carefully designed wrappers.

## Device state machine

Every driver should make initialization explicit:

```text
reset
  → detect/present
  → acknowledge
  → identify driver
  → negotiate features
  → allocate/configure queues
  → mark driver ready
  → operate
  → quiesce
  → reset
```

Reject impossible transitions. If initialization fails, reset the device and release all resources. Do not leave half-owned queue memory behind.

## Ownership model

For each request buffer, ownership moves through states:

```text
free
  → driver preparing
  → published to device
  → device owned/in flight
  → completion visible
  → driver reclaiming
  → free
```

The driver must not mutate or free device-owned memory. The device's completion data is untrusted until validated.

## Interrupt versus polling

Polling is useful for initial correctness because it removes interrupt-routing variables. Once the data path works:

1. enable interrupts;
2. acknowledge sources correctly;
3. process bounded batches;
4. suppress or coalesce notifications where supported;
5. retain a polling fallback for debugging.

High packet rates can produce interrupt storms. A hybrid model can disable interrupts while a queue is busy, poll a bounded batch, then re-enable when idle.

## Device-independent interfaces

Keep library OS services independent of transport details:

```rust
trait NetDevice {
    fn receive(&mut self) -> Option<RxToken>;
    fn transmit(&mut self) -> Option<TxToken>;
}

trait BlockDevice {
    fn submit(&mut self, request: BlockRequest) -> Result<RequestId, Error>;
    fn poll_complete(&mut self) -> Option<BlockCompletion>;
}
```

The network stack should not know whether the device is virtio-MMIO, virtio-PCI, a hosted TAP adapter, or a test fake.

## Host-tested fake devices

Build device models in ordinary Rust tests. A fake can:

- expose registers;
- observe driver state transitions;
- consume descriptors;
- inject malformed completions;
- delay or reorder operations;
- reset at adversarial points;
- count notifications.

This lets you test queue logic without a VM and later reuse the model in `oc-vmm`.

## Defensive validation

Treat device output as untrusted because a compromised or buggy VMM could return malformed data. Validate:

- queue indexes;
- descriptor identifiers;
- chain length and loops;
- reported byte counts;
- status values;
- buffer boundaries;
- completion of an actually in-flight request;
- feature-dependent fields.

A compromised guest should also be assumed capable of sending malicious device requests to the host. Defensive engineering is required on both sides.

## Reset and shutdown

Reset is not merely reinitialization. It must define what happens to in-flight buffers, interrupts, pending completions, and task waiters. On quiesce:

```text
stop accepting new requests
drain or cancel in-flight requests
flush where required
mask/disable notifications
record stable device state
```

If cancellation is impossible, snapshot must wait for a bounded drain or fail cleanly.

## Debugging playbook

### Device reports ready but no completions arrive

Check feature negotiation, queue addresses, queue size, descriptor flags, notification register, GPA/GVA confusion, and memory publication ordering.

### First request works, second corrupts memory

Check descriptor reuse, ring index wrap-around, completion reclamation, queue-free list, and whether buffers remain live.

### Interrupt fires continuously

Check acknowledgement semantics, status bits, queue draining, masking order, and whether the device level-trigger condition remains asserted.

### Polling works but interrupts do not

Keep the queue code unchanged and isolate interrupt routing: vector configuration, injection, IDT entry, APIC acknowledgement, and source masking.

## Exercises

1. Implement a toy MMIO device in userspace and a driver against it.
2. Add a fake DMA engine that writes after a random delay and detect premature buffer reuse.
3. Inject a descriptor loop and verify bounded rejection.
4. Reset a device after every initialization transition and check resource cleanup.
5. Compare polling, one-interrupt-per-completion, and batched completion processing.
6. Write a driver contract document for one device before implementing it.

## Review questions

1. Why is volatile access not the same as an atomic access?
2. Which address domain belongs in a DMA descriptor?
3. When exactly does ownership transfer to a device?
4. Why should polling be implemented before interrupts?
5. What state must reset invalidate?
6. Why must both guest drivers and VMM device models validate input?

## Opencomputer connection

Virtio is part of Opencomputer's tenant boundary. A malicious guest can construct hostile descriptor chains, while a malicious or compromised device model can return hostile completions. The runtime should minimize device count, sandbox the VMM, fuzz both sides, meter queue activity, and make device topology part of the signed workload and snapshot identity.
