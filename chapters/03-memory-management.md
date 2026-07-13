# Chapter 3 — Physical Memory, Virtual Memory, and Allocation

## Purpose

Memory management connects the linker, bootloader, CPU, allocator, drivers, application, and hypervisor. Most catastrophic early-kernel bugs are memory-contract bugs: an address is interpreted in the wrong domain, a page-table bit is wrong, an allocator returns overlapping memory, or a DMA buffer is not visible to the device as expected.

## Learning objectives

You should be able to:

- distinguish virtual, guest-physical, host-virtual, and host-physical addresses;
- walk an x86-64 page-table hierarchy;
- create mappings with explicit permissions and ownership;
- implement frame, heap, stack, and DMA allocation;
- explain TLB behavior and invalidation;
- design guard pages and direct-physical mappings;
- define memory invariants that survive snapshot and restore.

## Address domains

In a virtualized system, at least four address domains may be relevant:

```text
guest virtual address (GVA)
    ↓ guest page tables

guest physical address (GPA)
    ↓ EPT/NPT managed by KVM

host physical address (HPA)

host virtual address (HVA)
    ↕ VMM process mappings of guest RAM
```

The VMM usually manipulates guest memory through an HVA corresponding to a GPA. The guest driver puts GPAs into virtio descriptors. The guest application normally uses GVAs. These domains must not be represented by the same naked integer type.

## Page-table hierarchy

For conventional four-level x86-64 paging, a virtual address is divided into indexes plus a page offset:

```text
| sign extension | PML4 | PDPT | PD | PT | offset |
```

Each entry contains an address and flags. Effective permissions are constrained by all levels in the walk. A writable leaf under a read-only parent remains read-only. Execute-disable behavior similarly composes through the hierarchy.

Your page-table API should make intent explicit:

```rust
map_page(
    virtual_page,
    physical_frame,
    PagePermissions::READ | PagePermissions::EXECUTE,
    MappingOwner::KernelImage,
)
```

Avoid a generic `u64 flags` argument scattered throughout the codebase.

## Recommended virtual layout

A simple first layout might include:

```text
low canonical range
├── null guard / intentionally unmapped
├── optional identity bootstrap mappings
└── MMIO window if needed

high canonical range
├── kernel image
├── read-only manifest/config
├── direct physical map
├── heap
├── task stacks with guard pages
└── temporary mapping window
```

The exact addresses are less important than documenting invariants and preventing accidental overlap.

## Physical-frame allocator

Start with a memory map from the boot environment. Normalize overlapping and adjacent entries, reserve everything already in use, and then expose frames through a typed allocator.

Implementation progression:

1. **Early bump allocator** for bootstrap page tables and metadata.
2. **Bitmap allocator** for simplicity and strong validation.
3. Optional **buddy allocator** if large contiguous allocations or scalability require it.
4. Per-CPU caches only after SMP and measurement justify them.

Required checks:

- no frame allocated twice;
- reserved ranges never returned;
- alignment and range arithmetic checked;
- free rejects unknown or already-free frames;
- metadata itself is reserved;
- allocation statistics remain consistent.

## Heap allocation

The heap allocator should be replaceable. A first implementation can use a linked-list allocator or a maintained allocator adapted to your environment. Separate the concerns:

```text
virtual-range allocator
    ↓
page mapper
    ↓
physical-frame allocator
    ↓
heap suballocator
```

Do not make the heap responsible for arbitrary page-table manipulation.

## Stacks

Allocate stacks as regions with at least one unmapped guard page. Record bounds in task metadata so crash reporting can detect whether RSP is plausible.

```text
higher addresses
┌─────────────────────┐
│ mapped stack pages  │
├─────────────────────┤
│ unmapped guard page │
└─────────────────────┘
lower addresses
```

Consider a second guard at the high end if stack direction or corruption risks justify it.

## DMA memory

A DMA allocation contract must define:

- guest-physical contiguity requirements;
- alignment;
- device-visible address;
- ownership transfer;
- cache-coherency assumptions;
- pinning/lifetime;
- whether the memory may be reclaimed while a request is in flight.

With virtio in a typical coherent x86 VM, cache maintenance is simpler than on noncoherent hardware, but compiler and CPU ordering still matter. The queue and buffer must remain alive until the device returns ownership.

## Permissions and hardening

Enforce permissions from the first working page-table implementation:

```text
kernel code       R-X
read-only data    R--
writable data     RW-
heap              RW-
stacks            RW- plus guard pages
DMA               RW-
configuration     R-- after initialization
page zero         unmapped
```

If writable executable memory is temporarily necessary during relocation or JIT-like setup, make the transition explicit, narrow, and audited. A first unikernel should normally avoid it entirely.

## TLB behavior

Changing a page-table entry does not automatically guarantee every CPU stops using cached translation state. On a single vCPU, invalidate the affected page or reload the address-space root according to the architectural requirements. With multiple vCPUs, you need a shootdown protocol. That complexity is one reason to defer SMP.

## Snapshot implications

A memory snapshot preserves allocator metadata and page contents. Restore must use the same image/configuration contract and device topology. If the guest stores host-derived values such as wall-clock offsets, external connection state, or entropy pools, the restore path must explicitly repair or refresh them.

## Debugging playbook

### Page fault after enabling your own tables

Check:

1. Is the currently executing instruction mapped?
2. Is the stack mapped through the switch?
3. Is the page-table root physical address correct?
4. Are intermediate entries present?
5. Are reserved bits zero?
6. Is the address canonical?
7. Is NX enabled and consistent with permissions?

### Heap corruption

Reduce to one allocator, enable red zones/canaries, poison freed memory, log allocations by call site, and run hosted randomized tests. Do not debug the allocator only inside QEMU.

### Virtio sees garbage

Verify the address placed in the descriptor is a GPA, not a GVA or HVA. Confirm the buffer is mapped, aligned, initialized, and published before notification.

## Exercises

1. Write a host-side page-table simulator and compare it with live guest walks.
2. Build a bitmap frame allocator with property tests for random allocate/free sequences.
3. Map the same physical frame at two virtual addresses and demonstrate aliasing.
4. Enforce read-only `.rodata`, then deliberately attempt a write and verify the fault report.
5. Add stack guard pages and produce a deterministic overflow test.
6. Implement a temporary mapping window for inspecting arbitrary physical frames.

## Review questions

1. Why does a VMM normally provide a GPA to HVA mapping rather than a GVA mapping?
2. How do permissions at intermediate page-table levels affect a leaf mapping?
3. Why should allocator metadata itself be reserved?
4. What must remain valid while a DMA request is in flight?
5. Why does adding a second vCPU complicate page-table updates?
6. Which memory state must be refreshed after cloning a snapshot?

## Opencomputer connection

Opencomputer should treat requested guest memory, image load ranges, block mappings, and snapshot memory as one validated plan. The worker must reject overlapping or impossible regions before creating the VM. Memory accounting should distinguish committed guest RAM, VMM overhead, shared immutable pages, snapshot working set, and host page-cache effects so scheduling decisions use real—not nominal—cost.
