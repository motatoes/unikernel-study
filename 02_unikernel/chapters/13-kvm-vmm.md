# Chapter 13 — Building a Small KVM VMM

## Purpose

A small VMM turns virtualization concepts into concrete code. You will allocate guest memory, create vCPUs, establish x86 state, enter `KVM_RUN`, handle exits, emulate devices, inject interrupts, and serialize state.

The target is educational but real:

```text
oc-vmm
├── KVM system/VM/vCPU wrappers
├── guest memory map
├── ELF loader
├── x86 boot-state builder
├── vCPU run loop
├── event loop
├── MMIO bus
├── toy and virtio devices
├── metrics
└── snapshot encoder/decoder
```

## Learning objectives

You should be able to:

- use `/dev/kvm` and query API/capabilities;
- create a VM, register guest memory, and create a vCPU;
- map and interpret the shared `kvm_run` structure;
- enter real mode for a tiny guest, then long mode for `oc-uk`;
- load ELF segments safely into guest memory;
- handle I/O, MMIO, HLT, shutdown, and internal-error exits;
- integrate vCPU execution with eventfds and epoll;
- implement a minimal device bus and interrupt path.

## KVM object model

KVM operations are scoped to file descriptors:

```text
/dev/kvm fd
  ├── system capability queries
  └── KVM_CREATE_VM
          ↓
       VM fd
        ├── memory regions
        ├── IRQ/chip/device configuration
        └── KVM_CREATE_VCPU
                ↓
             vCPU fd
              ├── register state
              ├── CPUID/MSR state
              └── KVM_RUN
```

Always check the API version and required `KVM_CAP_*` capabilities. Capabilities, not host-kernel version strings, are the supported feature contract.

## Minimal creation sequence

A simplified flow:

```text
open /dev/kvm
KVM_GET_API_VERSION
KVM_CHECK_EXTENSION...
KVM_CREATE_VM
allocate anonymous guest RAM
KVM_SET_USER_MEMORY_REGION
KVM_CREATE_VCPU
KVM_GET_VCPU_MMAP_SIZE
mmap shared kvm_run area
set initial registers
loop KVM_RUN and handle exits
```

Wrap each resource in RAII types and make partial-construction cleanup automatic.

## Guest memory

Represent guest RAM as one or more nonoverlapping slots:

```rust
struct MemoryRegion {
    guest_base: GuestPhysAddr,
    len: usize,
    host_ptr: NonNull<u8>,
    slot: u32,
    flags: MemoryFlags,
}
```

Provide checked translation:

```rust
fn get_slice(
    &self,
    guest_addr: GuestPhysAddr,
    len: usize,
) -> Result<&[u8], GuestMemoryError>;
```

Never add a guest-provided offset to a host pointer without overflow and bounds checks.

## First guest: tiny real-mode code

Begin with a few bytes that write to an emulated I/O port and halt. This isolates KVM setup from ELF and long-mode complexity.

Exit-loop sketch:

```rust
loop {
    match vcpu.run()? {
        Exit::IoOut { port, data } => io_bus.write(port, data)?,
        Exit::IoIn { port, data } => io_bus.read(port, data)?,
        Exit::MmioRead { address, data } => mmio_bus.read(address, data)?,
        Exit::MmioWrite { address, data } => mmio_bus.write(address, data)?,
        Exit::Hlt => break,
        Exit::Shutdown => return Err(VmError::GuestTripleFault),
        Exit::InternalError(info) => return Err(VmError::KvmInternal(info)),
        other => return Err(VmError::UnsupportedExit(other)),
    }
}
```

The exact Rust API depends on whether you use raw ioctls or a wrapper. Implement the raw path once, then refactor.

## ELF loading

Reuse the strict loader developed earlier. Validate every segment before copying any data:

1. correct architecture/class/endianness;
2. `filesz <= memsz`;
3. checked file range;
4. checked guest-memory range;
5. no overlap with reserved boot structures or device MMIO;
6. permitted alignment;
7. entry point lies in executable load range;
8. zero-fill memory tail.

Use a two-phase loader: validate the whole plan, then mutate guest memory. This prevents half-loaded images after a late error.

## Entering long mode

To start an x86-64 guest without firmware, the VMM must construct enough state for long mode. That normally includes:

- page tables in guest RAM;
- GDT in guest RAM;
- control-register values;
- EFER long-mode bits;
- segment state compatible with long mode;
- RIP at the kernel entry;
- RSP at a mapped stack;
- boot-information address in the agreed register or memory location.

Document the boot ABI in one place. Do not scatter magic addresses through the loader and guest.

## CPUID and CPU contract

A snapshot or portable image should not depend on every host feature. Define a virtual CPU contract and expose only the features your guest expects. At minimum, record:

- vendor/model policy or baseline;
- required long-mode, NX, APIC, and timing features;
- XSAVE/SIMD policy;
- invariant/virtual TSC assumptions;
- physical/virtual address widths.

Start conservatively. Host-passthrough is easy for local development but creates migration and snapshot compatibility problems.

## Event loop

A simple one-vCPU VMM can run the vCPU in its own thread and coordinate through:

- eventfd for device notifications and shutdown;
- epoll for TAP, vsock/backend sockets, timers, and control requests;
- a bounded channel for vCPU/device actions;
- atomic state for pause/stop requests.

Avoid executing arbitrary API work on the vCPU thread. Define clear ownership for VM state.

## MMIO bus

Represent address ranges explicitly:

```rust
trait MmioDevice {
    fn read(&mut self, offset: u64, data: &mut [u8]) -> Result<(), DeviceError>;
    fn write(&mut self, offset: u64, data: &[u8]) -> Result<(), DeviceError>;
}
```

The bus validates width and range, finds one device, calculates offset, and records metrics. Reject overlapping registrations.

Begin with a toy device before virtio. It should expose a few registers, trigger one interrupt, and serialize its state.

## Interrupt injection

For a minimal design, configure the required virtual interrupt-controller machinery through KVM or implement the simplest supported injection path for your guest. The device model should raise a logical interrupt; the interrupt controller decides how and when it reaches the vCPU.

Keep device state level-aware. Clearing an injected interrupt without clearing the underlying completion condition can cause lost or repeated interrupts.

## VMM metrics

At minimum collect:

```text
VM exits by reason
exit handling duration
vCPU run time
MMIO reads/writes by device
I/O port operations
interrupt injections
queue notifications
bytes/packets/block operations
pause and snapshot duration
fatal exit reason
```

Metrics are part of debugging, not post-production polish.

## Debugging playbook

### `KVM_RUN` returns immediately in a loop

Classify the exit reason. Common causes include unhandled port polling, MMIO reads returning an invalid value, a pending shutdown, or incorrect interrupt state.

### Guest triple faults during long-mode entry

Check page tables, CR0/CR4/EFER order, GDT descriptors, segment state, canonical RIP/RSP, and the first guest instruction. Capture sregs and disassembly before restarting.

### Memory appears correct from the host but guest faults

Verify the GPA is actually mapped by KVM, guest page tables refer to the right GPAs, and the VMM did not confuse file offset, GPA, or HVA.

### vCPU cannot be stopped cleanly

Use a defined kick mechanism rather than waiting for a guest exit indefinitely. Coordinate pause state and ensure all vCPU threads acknowledge a barrier before snapshot.

## Exercises

1. Run a real-mode guest that writes characters through port I/O.
2. Add an ELF plan validator and malicious-image tests.
3. Boot `oc-kernel` directly into long mode.
4. Add a toy MMIO device and interrupt.
5. Produce a VM-exit histogram for the HTTP unikernel.
6. Replace raw KVM wrappers selectively with current rust-vmm components after the raw implementation works.

## Review questions

1. Why does KVM use several file-descriptor classes?
2. Why should ELF validation finish before guest memory mutation begins?
3. Which state must the VMM initialize for direct long-mode boot?
4. Why is host CPU passthrough problematic for portable snapshots?
5. What work belongs on the vCPU thread?
6. How does a logical device interrupt differ from injection into a vCPU?

## Opencomputer connection

`oc-vmm` gives Opencomputer a reference implementation of the exact VM contract it needs. Even when production uses Firecracker, this reference can validate image loading, boot ABI, device behavior, snapshots, and performance assumptions independently of a larger VMM.
