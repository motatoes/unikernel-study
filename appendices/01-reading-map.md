# Appendix A — Reading Map and Primary Sources

> Links and version-sensitive references in this appendix were checked on 2026-07-13. Pin local copies or commits for a long-running course because websites and repository layouts change.

## How to read

Use three passes:

1. **Preview** — identify vocabulary, diagrams, and questions.
2. **Implementation pass** — read only the sections required by the current lab and annotate the source/code path.
3. **Review pass** — return after implementation and record what the text made clear only in hindsight.

Specifications should be read by feature, not cover to cover.

## Core systems books

### Computer Systems: A Programmer's Perspective, third edition

Official site: <https://csapp.cs.cmu.edu/>

Priority chapters:

| Chapter | Topic | Course use |
|---:|---|---|
| 1 | Tour of computer systems | Week 1 orientation |
| 2 | Information representation | address types, overflow, endian handling |
| 3 | Machine-level programs | x86-64 assembly and ABI |
| 6 | Memory hierarchy | cache behavior and queue performance |
| 7 | Linking | ELF, symbols, relocations, static composition |
| 8 | Exceptional control flow | traps, context, process mechanics |
| 9 | Virtual memory | address translation and allocators |
| 10 | System I/O | compatibility layer |
| 11 | Network programming | application networking model |
| 12 | Concurrency | threads, synchronization, races |

Do not skip Chapter 7. The linker is part of the unikernel architecture.

### Operating Systems: Three Easy Pieces

Official site: <https://pages.cs.wisc.edu/~remzi/OSTEP/>

Read:

```text
CPU virtualization: 2, 4–10
Memory:             13–20, selected 21–23
Concurrency:        26–33
I/O/persistence:    36, 39, 40, 42
```

Use the free chapter PDFs and simulation/measurement exercises. For each chapter, answer the “crux” question before reading the solution.

### xv6 and MIT 6.1810

Current course/xv6 landing page: <https://pdos.csail.mit.edu/6.828/xv6>

Current lab sequence used by this handbook:

```text
utilities
system calls
page tables
traps
copy-on-write
network driver
locks
file system
mmap
```

Read the xv6 book beside the source. The important skill is tracing complete paths, not memorizing files.

### Writing an OS in Rust

Official series: <https://os.phil-opp.com/>

Use the maintained second-edition sequence:

```text
freestanding binary
minimal kernel
VGA/printing exercise
testing
CPU exceptions
double faults
hardware interrupts
paging introduction
paging implementation
heap allocation
allocator designs
async/await
```

Adapt it: make serial canonical, retain the test infrastructure, and refactor tutorial code behind your own interfaces.

## Rust references

- Rust Book: <https://doc.rust-lang.org/book/>
- Rust Reference: <https://doc.rust-lang.org/reference/>
- Rustonomicon: <https://doc.rust-lang.org/nomicon/>
- Embedded Rust `no_std` overview: <https://docs.rust-embedded.org/book/intro/no-std.html>

Priority topics:

```text
ownership and lifetimes
traits and generics
smart pointers
concurrency and atomics
async/await
unsafe Rust
layout and representation
inline assembly
undefined behavior
Send and Sync
```

## OSDev Wiki track

Start:

- <https://wiki.osdev.org/Introduction>
- <https://wiki.osdev.org/Getting_Started>
- <https://wiki.osdev.org/Required_Knowledge>
- <https://wiki.osdev.org/Beginner_Mistakes>
- <https://wiki.osdev.org/What_Order_Should_I_Make_Things_In%3F>
- <https://wiki.osdev.org/Testing>

Toolchain and binary:

- <https://wiki.osdev.org/GCC_Cross-Compiler>
- <https://wiki.osdev.org/Rust>
- <https://wiki.osdev.org/Boot_Sequence>
- <https://wiki.osdev.org/Limine_Bare_Bones>
- <https://wiki.osdev.org/Higher_Half_Kernel>
- <https://wiki.osdev.org/ELF>
- <https://wiki.osdev.org/Linker_Scripts>
- <https://wiki.osdev.org/System_V_ABI>
- <https://wiki.osdev.org/Serial_Ports>

CPU and interrupts:

- <https://wiki.osdev.org/Global_Descriptor_Table>
- <https://wiki.osdev.org/Interrupt_Descriptor_Table>
- <https://wiki.osdev.org/Interrupt_Service_Routines>
- <https://wiki.osdev.org/Exceptions>
- <https://wiki.osdev.org/APIC>
- <https://wiki.osdev.org/APIC_Timer>
- <https://wiki.osdev.org/HPET>

Memory and execution:

- <https://wiki.osdev.org/Memory_Management>
- <https://wiki.osdev.org/Page_Tables>
- <https://wiki.osdev.org/Page_Frame_Allocation>
- <https://wiki.osdev.org/Context_Switching>
- <https://wiki.osdev.org/Scheduling_Algorithms>
- <https://wiki.osdev.org/Spinlock>

Devices and networking:

- <https://wiki.osdev.org/PCI>
- <https://wiki.osdev.org/PCI_Express>
- <https://wiki.osdev.org/Virtio>
- <https://wiki.osdev.org/Network_Stack>
- <https://wiki.osdev.org/ARP>
- <https://wiki.osdev.org/ICMP>
- <https://wiki.osdev.org/User%3ADemindiro/Networking_Tutorial>

Debugging and support:

- <https://wiki.osdev.org/Debugging>
- <https://wiki.osdev.org/Stack_Trace>
- <https://wiki.osdev.org/Random_Number_Generator>
- <https://wiki.osdev.org/Time_And_Date>

OSDev pages vary in depth and age. Use the linked primary specifications for critical details.

## Architecture and ABI specifications

### Intel x86-64

Intel Software Developer Manuals: <https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html>

Read by topic:

```text
long mode and control registers
paging and protection
exceptions and interrupts
GDT/IDT/TSS
APIC and multiprocessor operation
memory ordering
VMX
```

### AMD64

AMD developer guides/manuals: <https://www.amd.com/en/search/documentation/hub.html>

Use for AMD-specific SVM, NPT, interrupt, and architectural comparisons.

### x86-64 System V ABI

Canonical project: <https://gitlab.com/x86-psABIs/x86-64-ABI>

Priority:

```text
data representation
calling sequence
stack alignment
register classes
ELF conventions
process initialization
```

### ELF and linker

- ELF gABI: <https://gabi.xinuos.com/>
- GNU ld manual: <https://sourceware.org/binutils/docs/ld/>
- LLVM lld documentation: <https://lld.llvm.org/>

## Virtualization sources

### KVM

Official API: <https://docs.kernel.org/virt/kvm/api.html>

Read:

```text
general fd/ioctl model
capability discovery
KVM_CREATE_VM
KVM_SET_USER_MEMORY_REGION
KVM_CREATE_VCPU
kvm_run
x86 register/CPUID state
interrupt and device interfaces
```

Related testing references:

- KVM selftests in the Linux source tree
- kvm-unit-tests: <https://gitlab.com/kvm-unit-tests/kvm-unit-tests>

### Virtio

Current course reference: Virtio 1.4 Committee Specification 01, published 2026-04-08:

<https://docs.oasis-open.org/virtio/virtio/v1.4/cs01/virtio-v1.4-cs01.html>

Read in order:

1. terminology and normative language;
2. device status and feature negotiation;
3. split virtqueues;
4. MMIO transport;
5. network device;
6. block device;
7. socket device;
8. packed queues and advanced features later.

### Firecracker

- Repository and documentation: <https://github.com/firecracker-microvm/firecracker>
- NSDI paper: <https://www.usenix.org/conference/nsdi20/presentation/agache>

Use the documentation in the pinned release/commit for:

```text
design and threat model
kernel/rootfs setup
networking
block devices
vsock
jailer/production host setup
snapshot support and versioning
API specification
```

### rust-vmm

Organization: <https://github.com/rust-vmm>

Community overview: <https://github.com/rust-vmm/community>

As of 2026, some formerly separate utility repositories have moved or are moving into a rust-vmm monorepo. Pin exact crate versions/commits and follow repository migration notices rather than relying on old tutorial paths.

Useful components include KVM bindings/wrappers, guest memory, loader utilities, event utilities, and virtio infrastructure.

## Networking sources

- smoltcp: <https://github.com/smoltcp-rs/smoltcp>
- IETF RFC index: <https://www.rfc-editor.org/>

Core protocol references:

```text
Ethernet II framing: IEEE material and implementation references
ARP: RFC 826
IPv4: RFC 791 plus current updates
ICMPv4: RFC 792 plus current updates
UDP: RFC 768
TCP: RFC 9293
TCP congestion control: relevant current RFCs
DNS: RFC 1034/1035 plus updates
```

Always check the RFC Editor page for updates/obsoletions.

## Unikernel and library-OS papers

Recommended sequence:

1. **Exokernel: An Operating System Architecture for Application-Level Resource Management**  
   <https://pdos.csail.mit.edu/6.828/2008/readings/engler95exokernel.pdf>

2. **Rethinking the Library OS from the Top Down**  
   <https://www.microsoft.com/en-us/research/publication/rethinking-the-library-os-from-the-top-down/>

3. **Unikernels: Library Operating Systems for the Cloud**  
   <https://anil.recoil.org/papers/2013-asplos-mirage.pdf>

4. **ClickOS and the Art of Network Function Virtualization**  
   <https://www.usenix.org/conference/nsdi14/technical-sessions/presentation/martins>

5. **Jitsu: Just-In-Time Summoning of Unikernels**  
   <https://anil.recoil.org/papers/2015-nsdi-jitsu.pdf>

6. **Unikraft: Fast, Specialized Unikernels the Easy Way**  
   <https://arxiv.org/abs/2104.12721>

7. **Unikernels as Processes**  
   <https://dl.acm.org/doi/10.1145/3267809.3267845>

8. **Programming Unikernels in the Large via Functor Driven Development**  
   <https://anil.recoil.org/papers/2019-mirage-functors.pdf>

9. Security surveys and assessments relevant to your chosen systems.

## Source archaeology targets

Read narrow paths, not entire repositories:

| System | Inspect |
|---|---|
| Linux | x86 entry/traps, page tables, KVM UAPI, virtio ring helpers |
| xv6 | traps, VM, scheduler, sleep/wakeup, filesystem, NIC lab |
| QEMU | one simple MMIO device, virtio queue handling, ELF/kernel loader |
| Firecracker | machine config, vCPU loop, event manager, net/block/vsock, snapshot docs |
| rust-vmm | KVM wrappers, guest memory, loader, event utilities |
| Solo5 | bindings/tender ABI, manifest, boot and I/O surface |
| MirageOS | device-independent signatures, configuration graph, network app |
| Unikraft | library selection, allocator, scheduler, libc/syscall shim |
| smoltcp | device trait, interface poll, socket sets, timer/deadline behavior |

For every source-reading session, record:

```text
question
entry function/type
call/data path
invariants
error path
one design idea to adopt
one design idea not suitable for oc-uk
```
