# Unikernel Mastery Curriculum

> A 60-week, implementation-led curriculum for learning operating systems, virtualization, library operating systems, and unikernels—ending with a custom x86-64 unikernel that runs under QEMU/KVM, a small VMM of your own, and Firecracker.

---

## Progress

| Phase | Weeks | Theme | Status |
|---|---:|---|:---:|
| 0 | 1–2 | Orientation and systems laboratory | [ ] |
| 1 | 3–8 | Machine model, ABI, ELF, linking, atomics | [ ] |
| 2 | 9–20 | OS fundamentals with OSTEP and xv6 | [ ] |
| 3 | 21–30 | Bare-metal x86-64 kernel | [ ] |
| 4 | 31–38 | Virtio, networking, storage, host control | [ ] |
| 5 | 39–46 | Virtualization, KVM, and a small VMM | [ ] |
| 6 | 47–52 | Library OS and unikernel systems archaeology | [ ] |
| 7 | 53–60 | Build and deploy your own unikernel | [ ] |

**Started:** `YYYY-MM-DD`  
**Target completion:** `YYYY-MM-DD`  
**Current week:** `00`

---

## Table of contents

- [1. Goal](#1-goal)
- [2. How to use this curriculum](#2-how-to-use-this-curriculum)
- [3. What you will build](#3-what-you-will-build)
- [4. Initial technical decisions](#4-initial-technical-decisions)
- [5. Time commitment and weekly cadence](#5-time-commitment-and-weekly-cadence)
- [6. Core bookshelf and references](#6-core-bookshelf-and-references)
- [7. Curated OSDev Wiki reading track](#7-curated-osdev-wiki-reading-track)
- [8. Curriculum](#8-curriculum)
- [9. Testing requirements](#9-testing-requirements)
- [10. Scope control](#10-scope-control)
- [11. Graduation criteria](#11-graduation-criteria)
- [12. Weekly work-log template](#12-weekly-work-log-template)
- [13. Paper-review template](#13-paper-review-template)
- [14. Architecture-decision template](#14-architecture-decision-template)
- [15. Benchmark matrix](#15-benchmark-matrix)

---

# 1. Goal

By completing this curriculum, you should be able to:

- explain execution from an ELF entry point to application startup;
- write and debug x86-64 assembly across the System V ABI;
- configure page tables, exceptions, interrupts, timers, and CPU state;
- implement physical-frame, virtual-memory, heap, stack, and DMA allocators;
- design cooperative and preemptive execution models;
- implement a virtqueue and virtio-net driver;
- explain VMX/SVM, EPT/NPT, VM exits, interrupt injection, and device emulation;
- create a VM directly through `/dev/kvm`;
- write a small userspace KVM virtual-machine monitor;
- statically link an application with a library operating system;
- port one Rust application and one C application;
- design a narrow application ABI and host/platform ABI;
- implement a limited, explicitly documented POSIX compatibility layer;
- threat-model and harden a single-address-space system;
- benchmark boot time, memory use, network performance, and VM exits;
- snapshot and restore a quiesced unikernel safely;
- integrate the resulting artifact with an Opencomputer-style control plane.

The final system should resemble:

```text
Opencomputer control plane
        │
        ▼
QEMU/KVM | oc-vmm | Firecracker
        │
        ├── virtio-net
        ├── virtio-block
        ├── virtio-vsock
        └── serial/debug channel
                │
                ▼
         oc-uk unikernel ELF
        ┌─────────────────────────────┐
        │ Statically linked app       │
        ├─────────────────────────────┤
        │ Native app API              │
        │ Optional limited POSIX shim │
        ├─────────────────────────────┤
        │ TCP/IP │ FS │ executor      │
        ├─────────────────────────────┤
        │ Memory │ time │ entropy     │
        ├─────────────────────────────┤
        │ Virtio device drivers       │
        ├─────────────────────────────┤
        │ x86-64 platform layer       │
        └─────────────────────────────┘
```

---

# 2. How to use this curriculum

This is not a reading list. It is a sequence of experiments.

Use the following learning loop for every subsystem:

```text
1. Read a conceptual textbook chapter.
2. Read the corresponding OSDev overview.
3. Write the mechanism in your own words.
4. Read the authoritative specification section.
5. Define software invariants.
6. Implement the smallest useful version.
7. Add negative and failure tests.
8. Compare with an existing implementation.
9. Record what changed in your mental model.
```

Use resources for different purposes:

| Resource type | Purpose |
|---|---|
| Textbook | Explains concepts and abstractions |
| OSDev Wiki | Provides practical orientation and subsystem checklists |
| Specification | Defines authoritative layout and behaviour |
| Research paper | Explains architecture and trade-offs |
| Production source code | Shows real engineering compromises |
| Your implementation | Proves you understand the mechanism |

OSDev Wiki is an orientation and reference resource, not a substitute for architecture manuals or a copy-and-paste kernel tutorial.

## Repository workflow

Create one GitHub issue per week, using the template near the end of this file. Close the issue only after the week’s verification criteria pass.

Suggested labels:

```text
phase-0-lab
phase-1-machine
phase-2-os
phase-3-kernel
phase-4-virtio
phase-5-vmm
phase-6-unikernel-study
phase-7-oc-uk
reading
implementation
test
benchmark
blocked
```

Suggested branch and commit conventions:

```text
branch: week-03/address-and-layout
commit: week-03: add typed physical and virtual addresses
commit: week-03: test alignment and endian conversions
commit: week-03: document unsafe MMIO invariants
```

---

# 3. What you will build

Keep four main projects separate. The learning kernel may donate code to the final unikernel, but do not assume it will become the final architecture unchanged.

```text
unikernel-course/
├── README.md
├── UNIKERNEL_MASTERY_CURRICULUM.md
├── flake.nix
├── justfile
│
├── labs/
│   ├── abi-lab/
│   ├── elf-inspector/
│   ├── page-table-simulator/
│   ├── trap-frame-simulator/
│   ├── lockfree-ring/
│   └── network-protocol-lab/
│
├── xv6-labs/
│   └── xv6-riscv/
│
├── oc-kernel/
│   └── exploratory x86-64 kernel
│
├── oc-vmm/
│   └── userspace KVM VMM
│
├── oc-uk/
│   └── application-specific library OS
│
├── apps/
│   ├── hello/
│   ├── udp-echo/
│   ├── http-service/
│   └── kv-service/
│
├── docs/
│   ├── notes/
│   ├── weekly/
│   ├── paper-reviews/
│   ├── architecture-decisions/
│   ├── BOOT-ABI.md
│   ├── APPLICATION-ABI.md
│   ├── THREAT-MODEL.md
│   ├── SNAPSHOT-FORMAT.md
│   ├── PERFORMANCE.md
│   └── PORTING.md
│
├── tests/
├── fuzz/
└── benches/
```

## Project responsibilities

### `labs/`

Small host-side experiments that make one mechanism visible without booting a kernel.

Examples:

- ELF parser;
- x86-64 ABI exercises;
- page-table walker over simulated memory;
- trap-frame construction;
- lock-free ring buffer;
- packet parser and checksum tests.

### `xv6-labs/`

Your modified xv6 tree and lab notes. This is where you learn complete OS paths before eliminating abstractions in your unikernel.

### `oc-kernel/`

An exploratory x86-64 bare-metal kernel. Its purpose is to learn boot, paging, traps, interrupts, allocation, tasks, and drivers.

### `oc-vmm/`

A small userspace VMM built against KVM. Its purpose is to make guest memory, vCPU state, VM exits, interrupt injection, device models, and snapshots concrete.

### `oc-uk/`

The final application-specific library OS with a platform abstraction, application ABI, virtio drivers, networking, host control, and Opencomputer integration.

---

# 4. Initial technical decisions

Use these constraints until a concrete requirement proves that one should change.

| Decision | Initial choice |
|---|---|
| Production architecture | x86-64 |
| Teaching architecture | RISC-V through xv6 |
| Primary language | Rust |
| Low-level code | Small amount of x86-64 assembly |
| Runtime | `#![no_std]` |
| Foreign application interface | Stable C ABI |
| Address spaces | One |
| Initial vCPU count | One |
| Initial scheduler | Cooperative/event-driven |
| Learning-kernel boot | Limine |
| Primary VMM | QEMU/KVM |
| Secondary VMM | Firecracker |
| Device model | Virtio |
| Initial virtio transport | MMIO |
| Networking exercise | Handwritten Ethernet/ARP/IPv4/ICMP/UDP |
| Product network stack | smoltcp |
| Initial storage | Embedded read-only archive |
| Writable storage | Virtio-block |
| Host control | Virtio-vsock |
| Application linkage | Static |
| Compatibility | Native API first, limited POSIX later |
| Build environment | Nix |
| Artifact | ELF plus embedded manifest |

## Core architectural separation

Keep these layers separate from the beginning:

```text
application
    ↓
library OS interfaces
    ↓
runtime services
    ↓
device-independent driver interfaces
    ↓
virtio transport
    ↓
platform boot, CPU, and interrupt layer
    ↓
QEMU | oc-vmm | Firecracker | hosted Linux tests
```

Application and networking code should not directly depend on QEMU- or Firecracker-specific details.

---

# 5. Time commitment and weekly cadence

Recommended pace: **12–15 hours per week for 60 weeks**.

| Activity | Weekly time |
|---|---:|
| Textbook/specification reading | 3 hours |
| Source-code or paper reading | 1–2 hours |
| Implementation | 7–8 hours |
| Tests and debugging | 2 hours |
| Design notes/postmortem | 1 hour |

Every week must produce something observable:

```text
boot log
unit test
integration test
packet capture
fault dump
benchmark
source-level trace
architecture diagram
design decision
```

Do not measure progress by pages read. Measure it by whether you can reproduce, explain, test, and modify the mechanism.

---

# 6. Core bookshelf and references

## 6.1 Textbooks and guided courses

### Computer Systems: A Programmer’s Perspective, 3rd edition

- Site: <https://csapp.cs.cmu.edu/3e/home.html>
- Read deeply: Chapters 1, 2, 3, 6, 7, 8, 9.
- Read selectively later: Chapters 10, 11, 12.

Why it matters:

- machine representation;
- assembly and calling conventions;
- memory hierarchy;
- object files and linking;
- exceptional control flow;
- virtual memory;
- I/O, networking, and concurrency.

### Operating Systems: Three Easy Pieces

- Site and free chapters: <https://pages.cs.wisc.edu/~remzi/OSTEP/>

Core chapters:

- CPU virtualization: 2, 4–10;
- memory: 13–15, 17–20;
- concurrency: 26–33;
- I/O and persistence: 36, 39, 40, 42.

### xv6 book, source, and MIT labs

- xv6 resources: <https://pdos.csail.mit.edu/6.1810/>
- xv6 book PDF: <https://pdos.csail.mit.edu/6.828/2025/xv6/book-riscv-rev5.pdf>

Read xv6 Chapters 1–11 and complete labs covering:

```text
Unix utilities
system calls
page tables
traps
copy-on-write
network driver
locks
file system
mmap
```

### Writing an OS in Rust, second edition

- Site: <https://os.phil-opp.com/>

Follow in order:

1. A Freestanding Rust Binary
2. A Minimal Rust Kernel
3. VGA Text Mode
4. Testing
5. CPU Exceptions
6. Double Faults
7. Hardware Interrupts
8. Introduction to Paging
9. Paging Implementation
10. Heap Allocation
11. Allocator Designs
12. Async/Await

Use serial output as the canonical output channel, even when completing the VGA exercise.

### Rust language references

- Rust Book: <https://doc.rust-lang.org/book/>
- Rustonomicon: <https://doc.rust-lang.org/nomicon/>
- Rust Reference: <https://doc.rust-lang.org/reference/>
- Type layout: <https://doc.rust-lang.org/reference/type-layout.html>
- Inline assembly: <https://doc.rust-lang.org/reference/inline-assembly.html>
- Undefined behaviour: <https://doc.rust-lang.org/reference/behavior-considered-undefined.html>

Important topics:

- ownership and lifetimes;
- traits and generics;
- smart pointers;
- atomics and concurrency;
- async execution;
- `unsafe` contracts;
- data layout and FFI;
- `Send` and `Sync`;
- volatile access and raw pointers.

## 6.2 Architecture, ABI, executable, and linker references

- Intel Software Developer Manuals: <https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html>
- AMD documentation hub: <https://www.amd.com/en/search/documentation/hub.html>
- x86-64 System V ABI: <https://gitlab.com/x86-psABIs/x86-64-ABI>
- ELF gABI: <https://gabi.xinuos.com/>
- GNU linker manual: <https://sourceware.org/binutils/docs/ld/>

Use architecture manuals by topic rather than reading them linearly:

```text
long mode
control registers
paging
protection
exceptions
IDT and TSS
APIC
multiprocessor startup
memory ordering
VMX/SVM
EPT/NPT
```

## 6.3 Virtualization references

- KVM API: <https://docs.kernel.org/virt/kvm/api.html>
- QEMU GDB usage: <https://www.qemu.org/docs/master/system/gdb.html>
- rust-vmm community: <https://github.com/rust-vmm/community>
- Firecracker repository: <https://github.com/firecracker-microvm/firecracker>
- Firecracker design: <https://github.com/firecracker-microvm/firecracker/blob/main/docs/design.md>
- Firecracker kernel/rootfs setup: <https://github.com/firecracker-microvm/firecracker/blob/main/docs/rootfs-and-kernel-setup.md>
- Firecracker vsock: <https://github.com/firecracker-microvm/firecracker/blob/main/docs/vsock.md>
- Firecracker snapshots: <https://github.com/firecracker-microvm/firecracker/blob/main/docs/snapshotting/snapshot-support.md>

## 6.4 Virtio and networking references

- Virtio specification: <https://docs.oasis-open.org/virtio/virtio/v1.4/>
- smoltcp: <https://github.com/smoltcp-rs/smoltcp>
- IETF RFC index: <https://www.rfc-editor.org/>

Protocol references to consult during the networking phase:

- Ethernet II framing;
- ARP: RFC 826;
- IPv4: RFC 791 and modern updates;
- ICMPv4: RFC 792 and modern updates;
- UDP: RFC 768;
- TCP: RFC 9293;
- DHCP: RFC 2131 where needed.

## 6.5 Unikernel and library-OS papers

Read in this order:

1. **Exokernel: An Operating System Architecture for Application-Level Resource Management**  
   <https://pdos.csail.mit.edu/6.828/2008/readings/engler95exokernel.pdf>

2. **Rethinking the Library OS from the Top Down**  
   <https://www.microsoft.com/en-us/research/publication/rethinking-the-library-os-from-the-top-down/>

3. **Unikernels: Library Operating Systems for the Cloud**  
   <https://anil.recoil.org/papers/2013-asplos-mirage.pdf>

4. **ClickOS and the Art of Network Function Virtualization**  
   <https://www.usenix.org/conference/nsdi14/technical-sessions/presentation/martins>

5. **Jitsu: Just-In-Time Summoning of Unikernels**  
   <https://www.usenix.org/conference/nsdi15/technical-sessions/presentation/madhavapeddy>

6. **Unikraft: Fast, Specialized Unikernels the Easy Way**  
   <https://arxiv.org/abs/2104.12721>

7. **Unikernels as Processes**  
   <https://dl.acm.org/doi/10.1145/3267809.3267845>

8. **Programming Unikernels in the Large**  
   <https://arxiv.org/abs/1905.02529>

9. **A Security Perspective on Unikernels**  
   <https://arxiv.org/abs/1911.06260>

10. **Assessing Unikernel Security**  
    <https://people.cs.pitt.edu/~babay/courses/cs3551/projects/SCADA_Unikernel/NCC_Group-Assessing_Unikernel_Security.pdf>

## 6.6 Virtualization papers

1. **Formal Requirements for Virtualizable Third Generation Architectures**  
   <https://dl.acm.org/doi/10.1145/361011.361073>

2. **Xen and the Art of Virtualization**  
   <https://www.cl.cam.ac.uk/research/srg/netos/papers/2003-xensosp.pdf>

3. **KVM: the Linux Virtual Machine Monitor**  
   <https://www.kernel.org/doc/ols/2007/ols2007v1-pages-225-230.pdf>

4. **Firecracker: Lightweight Virtualization for Serverless Applications**  
   <https://www.usenix.org/conference/nsdi20/presentation/agache>

## 6.7 Optional deeper books

These are not prerequisites for the first MVP.

- *The Art of Multiprocessor Programming*, 2nd edition — read when adding SMP and high-performance concurrent structures.
- *Virtual Machines: Versatile Platforms for Systems and Processes* — conceptual virtualization depth.
- *Linkers and Loaders* by John Levine — deeper executable and linking knowledge.
- *TCP/IP Illustrated, Volume 1* — deeper protocol analysis.
- *Computer Architecture: A Quantitative Approach* — deeper architecture and performance analysis.

---

# 7. Curated OSDev Wiki reading track

Entry point: <https://wiki.osdev.org/Introduction>

## 7.1 Orientation

Read before writing kernel code:

- [Introduction](https://wiki.osdev.org/Introduction)
- [Getting Started](https://wiki.osdev.org/Getting_Started)
- [Required Knowledge](https://wiki.osdev.org/Required_Knowledge)
- [Beginner Mistakes](https://wiki.osdev.org/Beginner_Mistakes)
- [What Order Should I Make Things In?](https://wiki.osdev.org/What_order_should_I_make_things_in)
- [Testing](https://wiki.osdev.org/Testing)

## 7.2 Toolchain, boot, and executable formats

- [GCC Cross-Compiler](https://wiki.osdev.org/GCC_Cross-Compiler)
- [Rust](https://wiki.osdev.org/Rust)
- [Boot Sequence](https://wiki.osdev.org/Boot_Sequence)
- [Limine Bare Bones](https://wiki.osdev.org/Limine_Bare_Bones)
- [Higher Half Kernel](https://wiki.osdev.org/Higher_Half_Kernel)
- [ELF](https://wiki.osdev.org/ELF)
- [Linker Scripts](https://wiki.osdev.org/Linker_Scripts)
- [System V ABI](https://wiki.osdev.org/System_V_ABI)
- [Serial Ports](https://wiki.osdev.org/Serial_Ports)

## 7.3 CPU, exceptions, and interrupts

- [Global Descriptor Table](https://wiki.osdev.org/Global_Descriptor_Table)
- [Interrupt Descriptor Table](https://wiki.osdev.org/Interrupt_Descriptor_Table)
- [Interrupt Service Routines](https://wiki.osdev.org/Interrupt_Service_Routines)
- [Exceptions](https://wiki.osdev.org/Exceptions)
- [Interrupts Tutorial](https://wiki.osdev.org/Interrupts_Tutorial)
- [APIC](https://wiki.osdev.org/APIC)
- [APIC Timer](https://wiki.osdev.org/APIC_Timer)
- [PIT](https://wiki.osdev.org/Programmable_Interval_Timer)
- [HPET](https://wiki.osdev.org/HPET)

## 7.4 Memory

- [Memory Management](https://wiki.osdev.org/Memory_Management)
- [Page Tables](https://wiki.osdev.org/Page_Tables)
- [Page Frame Allocation](https://wiki.osdev.org/Page_Frame_Allocation)
- [Higher Half Kernel](https://wiki.osdev.org/Higher_Half_Kernel)
- [Heap](https://wiki.osdev.org/Heap)
- [Context Switching](https://wiki.osdev.org/Context_Switching)

Validate page-table, protection, and TLB behaviour against the Intel or AMD manuals.

## 7.5 Scheduling, locking, and SMP

Read only after the single-vCPU system is stable:

- [Scheduling Algorithms](https://wiki.osdev.org/Scheduling_Algorithms)
- [Context Switching](https://wiki.osdev.org/Context_Switching)
- [Spinlock](https://wiki.osdev.org/Spinlock)
- [Symmetric Multiprocessing](https://wiki.osdev.org/Symmetric_Multiprocessing)
- [Multiprocessor Startup](https://wiki.osdev.org/Multiprocessor_Startup)
- [MADT](https://wiki.osdev.org/MADT)
- [APIC](https://wiki.osdev.org/APIC)

## 7.6 Devices and discovery

- [PCI](https://wiki.osdev.org/PCI)
- [PCI Express](https://wiki.osdev.org/PCI_Express)
- [Virtio](https://wiki.osdev.org/Virtio)
- [ACPI](https://wiki.osdev.org/ACPI)
- [MADT](https://wiki.osdev.org/MADT)

Use the OASIS Virtio specification as the final authority for state transitions, queue layout, notification, and device-specific behaviour.

## 7.7 Networking

- [Network Stack](https://wiki.osdev.org/Network_Stack)
- [Ethernet](https://wiki.osdev.org/Ethernet)
- [ARP](https://wiki.osdev.org/Address_Resolution_Protocol)
- [Internet Protocol](https://wiki.osdev.org/Internet_Protocol)
- [ICMP](https://wiki.osdev.org/Internet_Control_Message_Protocol)
- [UDP](https://wiki.osdev.org/User_Datagram_Protocol)

Validate packet layouts and state machines against RFCs.

## 7.8 Time, entropy, and debugging

- [Time and Date](https://wiki.osdev.org/Time_And_Date)
- [Random Number Generator](https://wiki.osdev.org/Random_Number_Generator)
- [Serial Ports](https://wiki.osdev.org/Serial_Ports)
- [Stack Trace](https://wiki.osdev.org/Stack_Trace)
- [Debugging](https://wiki.osdev.org/Debugging)
- [Testing](https://wiki.osdev.org/Testing)

---

# 8. Curriculum

# Phase 0 — Orientation and systems laboratory

## Week 01 — Define the destination

**Objective:** Decide what you are building and prevent early scope drift.

### Reading

- [ ] CS:APP Chapter 1, *A Tour of Computer Systems*.
- [ ] OSDev: [Introduction](https://wiki.osdev.org/Introduction).
- [ ] OSDev: [Getting Started](https://wiki.osdev.org/Getting_Started).
- [ ] OSDev: [Required Knowledge](https://wiki.osdev.org/Required_Knowledge).
- [ ] OSDev: [Beginner Mistakes](https://wiki.osdev.org/Beginner_Mistakes).

### Implementation

- [ ] Create the repository structure in [What you will build](#3-what-you-will-build).
- [ ] Write `VISION.md` describing the final useful application.
- [ ] Write `NON_GOALS.md` using the scope list later in this document.
- [ ] Write `docs/architecture-decisions/ADR-0001-initial-scope.md`.
- [ ] Draw the final target stack from application to VMM.
- [ ] Choose a baseline comparison guest, such as a minimal NixOS or Alpine VM.

### Verification

- [ ] You can explain why the project is a library OS/unikernel rather than merely a small kernel.
- [ ] You can identify the intended security boundary.
- [ ] You have selected one first application: HTTP service, DNS service, or key-value service.
- [ ] You have committed the vision and non-goals.

### Exit artifact

```text
docs/VISION.md
docs/NON_GOALS.md
docs/architecture-decisions/ADR-0001-initial-scope.md
```

---

## Week 02 — Reproducible systems laboratory

**Objective:** Build a repeatable environment for compiling, booting, debugging, testing, and measuring systems code.

### Reading

- [ ] OSDev: [Testing](https://wiki.osdev.org/Testing).
- [ ] OSDev: [What Order Should I Make Things In?](https://wiki.osdev.org/What_order_should_I_make_things_in).
- [ ] QEMU: [GDB usage](https://www.qemu.org/docs/master/system/gdb.html).
- [ ] Nix language and flakes material required to define a development shell.

### Implementation

- [ ] Create `flake.nix` with Rust, LLVM, Clang, LLD, binutils, NASM, QEMU, GDB, Python, `tcpdump`, `socat`, and `just`.
- [ ] Add `just build`, `run`, `debug`, `test`, `inspect`, and `clean` targets.
- [ ] Create a script that starts QEMU with serial output captured to a file.
- [ ] Create a script that starts QEMU with `-s -S` and attaches GDB.
- [ ] Create a machine-readable success/failure convention for VM tests.
- [ ] Record baseline QEMU process startup and minimal Linux guest boot time.

### Verification

- [ ] A clean clone enters the same development shell.
- [ ] QEMU starts paused and GDB attaches before guest execution.
- [ ] Serial output is captured in CI-friendly form.
- [ ] Your baseline measurements are stored as JSON or CSV.

### Exit artifact

```text
flake.nix
justfile
scripts/run-qemu
scripts/debug-qemu
scripts/measure-boot
baselines/linux-qemu.json
```

## Phase 0 gate

- [ ] Reproducible development environment.
- [ ] Repeatable QEMU execution.
- [ ] GDB attach before first guest instruction.
- [ ] Machine-readable VM test result.
- [ ] Baseline comparison recorded.

---

# Phase 1 — Machine model, ABI, ELF, linking, and atomics

## Week 03 — Data representation and typed addresses

**Objective:** Make machine representation, layout, alignment, and address domains explicit.

### Reading

- [ ] CS:APP Chapter 2, *Representing and Manipulating Information*.
- [ ] Rust Book Chapters 4 and 10.
- [ ] Rustonomicon: data layout and unsafe foundations.
- [ ] OSDev: [Rust](https://wiki.osdev.org/Rust).
- [ ] OSDev: [System V ABI](https://wiki.osdev.org/System_V_ABI).

### Implementation

- [ ] Implement `PhysAddr`, `VirtAddr`, `GuestPhysAddr`, and `GuestVirtAddr` newtypes.
- [ ] Implement checked alignment helpers: `align_up`, `align_down`, and `is_aligned`.
- [ ] Implement endian wrapper types for little- and big-endian integers.
- [ ] Implement narrow `volatile_read` and `volatile_write` APIs.
- [ ] Define an aligned DMA buffer wrapper with a documented ownership model.
- [ ] Inspect `#[repr(C)]`, `#[repr(transparent)]`, and packed-layout behaviour.

### Verification

- [ ] Tests cover overflow, canonical addresses, alignment, and endian conversion.
- [ ] No public API uses a naked `usize` where the address domain matters.
- [ ] Every unsafe function contains a `# Safety` contract.
- [ ] You can explain padding and alignment in one device-register struct.

### Exit artifact

```text
labs/address-types/
docs/notes/week-03-data-layout.md
```

---

## Week 04 — x86-64 assembly and calling conventions

**Objective:** Cross language boundaries without guessing about stack or register state.

### Reading

- [ ] CS:APP Chapter 3, *Machine-Level Representation of Programs*.
- [ ] x86-64 psABI: function calling sequence, register classes, stack alignment, and process entry.
- [ ] Rust Reference: inline assembly.

### Implementation

- [ ] Write an assembly `_start` that calls Rust.
- [ ] Call assembly functions from Rust and Rust functions from assembly.
- [ ] Save and restore all callee-saved registers.
- [ ] Verify stack alignment before a function call.
- [ ] Implement a host-side `CpuContext` structure.
- [ ] Write one deliberately broken example that violates the ABI and document the symptom.

### Verification

- [ ] Disassembly matches your expected parameter and return-value placement.
- [ ] Stack alignment tests pass.
- [ ] Register-preservation tests pass.
- [ ] You can annotate every instruction in your startup stub.

### Exit artifact

```text
labs/abi-lab/
docs/notes/week-04-x86-64-abi.md
```

---

## Week 05 — ELF object files and executable loading

**Objective:** Understand the artifact that your bootloader and VMM will load.

### Reading

- [ ] CS:APP Chapter 7: relocatable files, symbols, relocation, executable files, and static libraries.
- [ ] ELF gABI: file header, program header, section header, symbols, and relocations.
- [ ] OSDev: [ELF](https://wiki.osdev.org/ELF).

### Implementation

- [ ] Write `elf-inspector` without relying on a high-level ELF library for the first version.
- [ ] Print ELF class, endianness, machine, entry point, and header offsets.
- [ ] Print loadable segments, file and memory sizes, alignment, and permissions.
- [ ] Print section names, symbols, and relocations.
- [ ] Detect overlapping or invalid loadable segments.
- [ ] Compare a static C binary, dynamic C binary, and Rust binary.

### Verification

- [ ] Inspector output agrees with `readelf` for selected binaries.
- [ ] Malformed-file tests do not panic or read out of bounds.
- [ ] You can explain sections versus segments.
- [ ] You can explain why `p_memsz` may exceed `p_filesz`.

### Exit artifact

```text
labs/elf-inspector/
docs/notes/week-05-elf.md
```

---

## Week 06 — Linker scripts and freestanding startup

**Objective:** Control the final image layout rather than accepting compiler defaults.

### Reading

- [ ] CS:APP Chapter 7 review.
- [ ] GNU linker manual: `ENTRY`, `SECTIONS`, output sections, symbols, alignment, `KEEP`, and program headers.
- [ ] OSDev: [Linker Scripts](https://wiki.osdev.org/Linker_Scripts).
- [ ] OSDev: [Limine Bare Bones](https://wiki.osdev.org/Limine_Bare_Bones).

### Implementation

- [ ] Create a freestanding binary with your own `_start`.
- [ ] Write a linker script with distinct `.text`, `.rodata`, `.data`, and `.bss` regions.
- [ ] Export linker-defined symbols for all region boundaries.
- [ ] Clear `.bss` explicitly at startup.
- [ ] Generate a linker map for every build.
- [ ] Add CI validation that rejects writable-executable load segments.

### Verification

- [ ] `readelf -l` shows the expected segment permissions.
- [ ] `.bss` is zeroed before Rust code depends on it.
- [ ] The entry point matches `_start`.
- [ ] The linker map explains every major output section.

### Exit artifact

```text
labs/freestanding-elf/
linker/x86_64.ld
scripts/check-elf-permissions
```

---

## Week 07 — Exceptions and trap-frame design

**Objective:** Understand asynchronous and exceptional control transfer before handling it on bare metal.

### Reading

- [ ] CS:APP Chapter 8, *Exceptional Control Flow*.
- [ ] Intel SDM topics: exception classes, error codes, interrupt gates, and privilege transitions.
- [ ] OSDev: [Exceptions](https://wiki.osdev.org/Exceptions).
- [ ] OSDev: [Interrupt Service Routines](https://wiki.osdev.org/Interrupt_Service_Routines).
- [ ] OSDev: [Interrupt Descriptor Table](https://wiki.osdev.org/Interrupt_Descriptor_Table).

### Implementation

- [ ] Build a host-side trap-frame simulator.
- [ ] Model exceptions with and without hardware error codes.
- [ ] Model nested exceptions and a double-fault condition.
- [ ] Define a stable register-dump format.
- [ ] Write a dispatcher that separates architecture entry from handler logic.

### Verification

- [ ] Tests cover at least divide error, invalid opcode, general protection, and page fault.
- [ ] The dispatcher cannot confuse vectors with and without error codes.
- [ ] You can draw the stack layout for same-privilege and privilege-changing entries.

### Exit artifact

```text
labs/trap-frame-simulator/
docs/notes/week-07-exceptions.md
```

---

## Week 08 — Atomics, locks, and ring buffers

**Objective:** Build the synchronization primitives later required by interrupts and virtqueues.

### Reading

- [ ] CS:APP Chapter 6 review for cache behaviour.
- [ ] OSTEP Chapter 28, *Locks*.
- [ ] Rustonomicon: atomics and `Send`/`Sync`.
- [ ] Linux kernel memory-barrier documentation.
- [ ] OSDev: [Spinlock](https://wiki.osdev.org/Spinlock).

### Implementation

- [ ] Implement a spin lock.
- [ ] Define an interrupt-safe locking strategy for future kernel use.
- [ ] Implement a bounded single-producer/single-consumer ring.
- [ ] Create a deliberately broken weak-ordering version.
- [ ] Add stress tests and model-based tests where practical.
- [ ] Document false sharing and cache-line alignment concerns.

### Verification

- [ ] No lost, duplicated, or reordered entries under stress.
- [ ] Acquire/release points are justified in comments.
- [ ] Lock tests include contention and recursive-acquisition failure.
- [ ] You can explain why a virtqueue requires publication ordering.

### Exit artifact

```text
labs/lockfree-ring/
labs/spinlock/
docs/notes/week-08-memory-ordering.md
```

## Phase 1 gate

- [ ] Freestanding ELF with custom entry point.
- [ ] Custom linker script and linker map.
- [ ] ELF inspector with malformed-input tests.
- [ ] Correct assembly/Rust ABI crossing.
- [ ] Typed address and MMIO primitives.
- [ ] Working ring buffer with documented memory ordering.

---

# Phase 2 — Operating-system fundamentals with OSTEP and xv6

## Week 09 — Processes and the Unix interface

**Objective:** Understand the abstractions your unikernel will intentionally remove.

### Reading

- [ ] OSTEP Chapters 2, 4, and 5.
- [ ] xv6 Chapter 1, *Operating system interfaces*.
- [ ] OSDev overview of [System Calls](https://wiki.osdev.org/System_Calls).

### Implementation

- [ ] Set up the xv6 tree and toolchain.
- [ ] Complete the Unix utilities lab.
- [ ] Trace one command from user entry through one system call.
- [ ] Record process creation, execution, waiting, and exit.

### Verification

- [ ] You can explain the purpose of `fork`, `exec`, `wait`, and file descriptors.
- [ ] You can identify which process abstractions the first unikernel will omit.
- [ ] Your xv6 changes pass the official tests.

### Exit artifact

```text
xv6-labs/week-09/
docs/notes/week-09-process-interface.md
```

---

## Week 10 — System-call entry and kernel organization

**Objective:** Trace a controlled transition from unprivileged code into a kernel.

### Reading

- [ ] OSTEP Chapter 6, *Limited Direct Execution*.
- [ ] xv6 Chapter 2.
- [ ] xv6 Sections 4.3–4.4 on system-call handling.
- [ ] OSDev: [Context Switching](https://wiki.osdev.org/Context_Switching).

### Implementation

- [ ] Complete the xv6 system-call lab.
- [ ] Add syscall tracing.
- [ ] Add system-call accounting by process and call number.
- [ ] Add invalid-pointer and invalid-length tests.
- [ ] Trace user registers through trampoline, trap frame, dispatch, and return.

### Verification

- [ ] Invalid userspace pointers fail safely.
- [ ] Traces show argument and return-value flow.
- [ ] You can explain why a single-address-space unikernel does not need the same syscall boundary.

### Exit artifact

```text
xv6-labs/week-10/
docs/notes/week-10-syscall-path.md
```

---

## Week 11 — Address spaces and page tables

**Objective:** Understand page-table structure and kernel/user mappings through a working OS.

### Reading

- [ ] OSTEP Chapters 13–15.
- [ ] xv6 Chapter 3, *Page tables*.
- [ ] OSDev: [Page Tables](https://wiki.osdev.org/Page_Tables).

### Implementation

- [ ] Complete the xv6 page-table lab.
- [ ] Add a page-table dump with permissions and physical targets.
- [ ] Trace one user virtual address through every page-table level.
- [ ] Identify kernel global mappings and per-process mappings.

### Verification

- [ ] You can explain PTE permission bits and address translation.
- [ ] You can distinguish a virtual address, physical address, and page-frame number.
- [ ] Dumped mappings match expected process layout.

### Exit artifact

```text
xv6-labs/week-11/
docs/notes/week-11-page-tables.md
```

---

## Week 12 — Allocation, page faults, and copy-on-write

**Objective:** Make demand allocation and copy-on-write concrete.

### Reading

- [ ] OSTEP Chapters 17–20.
- [ ] xv6 Chapter 5, *Page faults*.
- [ ] OSDev: [Page Frame Allocation](https://wiki.osdev.org/Page_Frame_Allocation).

### Implementation

- [ ] Complete the xv6 copy-on-write lab.
- [ ] Count page faults by cause.
- [ ] Count allocations, reference-count changes, and copied pages.
- [ ] Add tests for writes to shared pages and process exit.

### Verification

- [ ] No leaked shared frames after process exit.
- [ ] Writes cause copies only where required.
- [ ] You can explain the TLB consequences of changing a PTE.

### Exit artifact

```text
xv6-labs/week-12/
docs/notes/week-12-cow.md
```

---

## Week 13 — Traps and return paths

**Objective:** Read and modify a real trap entry/exit path.

### Reading

- [ ] xv6 Chapter 4, *Traps and system calls*.
- [ ] Revisit OSTEP Chapter 6.
- [ ] Revisit OSDev exception and ISR pages.

### Implementation

- [ ] Complete the xv6 traps lab.
- [ ] Annotate `trampoline.S` line by line.
- [ ] Add a controlled register dump.
- [ ] Instrument trap causes and return paths.

### Verification

- [ ] You can reconstruct the trap frame from assembly.
- [ ] You can explain which code runs with which page table active.
- [ ] Your annotations are stored in the repository.

### Exit artifact

```text
xv6-labs/week-13/
docs/notes/week-13-traps.md
```

---

## Week 14 — Devices, interrupts, and a network driver

**Objective:** Trace hardware-visible queue state into kernel network processing.

### Reading

- [ ] OSTEP Chapter 36, *I/O Devices*.
- [ ] xv6 Chapter 6, *Interrupts and device drivers*.
- [ ] OSDev: [Interrupts Tutorial](https://wiki.osdev.org/Interrupts_Tutorial).
- [ ] OSDev: [Network Stack](https://wiki.osdev.org/Network_Stack).

### Implementation

- [ ] Begin or complete the xv6 network-driver lab.
- [ ] Instrument descriptor ownership and ring indices.
- [ ] Correlate guest driver logs with host packet captures.
- [ ] Count interrupts, transmitted packets, received packets, and drops.

### Verification

- [ ] You can trace a packet from host interface to guest receiver.
- [ ] Ring ownership does not become ambiguous.
- [ ] Driver counters agree with packet capture within expected limits.

### Exit artifact

```text
xv6-labs/week-14/
docs/notes/week-14-driver-path.md
```

---

## Week 15 — Scheduling policy

**Objective:** Understand what a scheduler decides and what it does not.

### Reading

- [ ] OSTEP Chapters 7–9.
- [ ] xv6 Chapter 8, *Scheduling*.
- [ ] OSDev: [Scheduling Algorithms](https://wiki.osdev.org/Scheduling_Algorithms).

### Implementation

- [ ] Add scheduler telemetry.
- [ ] Implement a small priority, lottery, or weighted scheduler modification.
- [ ] Measure fairness and starvation behaviour.
- [ ] Create CPU-bound and I/O-bound test workloads.

### Verification

- [ ] Scheduler metrics reveal expected policy behaviour.
- [ ] You can explain context-switch cost and scheduling latency.
- [ ] You can state why the first unikernel starts with cooperative execution.

### Exit artifact

```text
xv6-labs/week-15/
docs/notes/week-15-scheduling.md
```

---

## Week 16 — Locking and contention

**Objective:** Identify kernel invariants protected by locks and measure contention.

### Reading

- [ ] OSTEP Chapters 26–30.
- [ ] xv6 Chapter 7, *Locking*.
- [ ] OSDev: [Spinlock](https://wiki.osdev.org/Spinlock).

### Implementation

- [ ] Complete the xv6 lock lab.
- [ ] Measure contention in allocator and buffer-cache locks.
- [ ] Add lock-order documentation.
- [ ] Create one forced deadlock or lock-order violation test in an isolated branch.

### Verification

- [ ] Lock contention is measurably reduced where the lab requires it.
- [ ] Every modified lock protects an explicit invariant.
- [ ] You can explain interrupt disabling around selected lock paths.

### Exit artifact

```text
xv6-labs/week-16/
docs/notes/week-16-locking.md
```

---

## Week 17 — Sleep, wake-up, and event-driven execution

**Objective:** Understand blocking, wake-ups, and the event-driven model that will shape the unikernel.

### Reading

- [ ] OSTEP Chapters 31–33.
- [ ] xv6 Chapter 9, *Sleep and wake-up*.
- [ ] OSDev: [Context Switching](https://wiki.osdev.org/Context_Switching).

### Implementation

- [ ] Trace xv6 sleep and wake-up paths.
- [ ] Build a Linux-hosted event reactor using `epoll`, timers, and explicit wake-ups.
- [ ] Add cancellation and deadline handling.
- [ ] Create a lost-wake-up test and document how correct ordering prevents it.

### Verification

- [ ] The reactor handles timer and I/O events without busy waiting.
- [ ] Lost-wake-up test fails in the broken version and passes in the correct version.
- [ ] You can compare threads, blocking, callbacks, and async tasks.

### Exit artifact

```text
labs/event-reactor/
docs/notes/week-17-events.md
```

---

## Week 18 — Filesystem implementation

**Objective:** Understand pathname lookup, inodes, caching, and block writes.

### Reading

- [ ] OSTEP Chapters 39 and 40.
- [ ] xv6 Chapter 10, *File system*.
- [ ] OSDev filesystem overview material as needed.

### Implementation

- [ ] Complete the xv6 filesystem lab.
- [ ] Trace pathname lookup to inode and block access.
- [ ] Instrument buffer-cache hits and misses.
- [ ] Document the write path and locking order.

### Verification

- [ ] You can trace create, write, sync, and close.
- [ ] You can explain inode, directory entry, and file descriptor differences.
- [ ] All official filesystem tests pass.

### Exit artifact

```text
xv6-labs/week-18/
docs/notes/week-18-filesystem.md
```

---

## Week 19 — Crash consistency and memory mapping

**Objective:** Learn the difference between memory correctness and durable-state correctness.

### Reading

- [ ] OSTEP Chapter 42, *Crash Consistency: FSCK and Journaling*.
- [ ] xv6 material required for the `mmap` lab.

### Implementation

- [ ] Complete the xv6 `mmap` lab.
- [ ] Inject simulated crashes around filesystem updates.
- [ ] Record which invariants survive.
- [ ] Compare filesystem freeze, journal commit, and application-level quiescing.

### Verification

- [ ] Mappings are released correctly on exit.
- [ ] Protection errors are handled correctly.
- [ ] Crash experiments have a written expected/observed matrix.

### Exit artifact

```text
xv6-labs/week-19/
docs/notes/week-19-crash-consistency.md
```

---

## Week 20 — xv6 capstone and OS model review

**Objective:** Demonstrate that you can modify a complete kernel subsystem and defend the design.

### Reading

- [ ] xv6 Chapter 11, *Concurrency revisited*.
- [ ] Review relevant OSTEP chapters for your capstone.

### Implementation

Choose one:

- [ ] priority or lottery scheduler;
- [ ] enhanced copy-on-write;
- [ ] network-driver enhancement;
- [ ] tracing subsystem;
- [ ] memory-mapped-file enhancement;
- [ ] filesystem consistency feature.

Write a design report covering:

- [ ] invariants;
- [ ] locking;
- [ ] failure cases;
- [ ] performance impact;
- [ ] tests;
- [ ] what changes on x86-64;
- [ ] which abstractions the unikernel can remove.

### Verification

- [ ] The capstone passes automated tests.
- [ ] The report includes a complete path diagram.
- [ ] You can defend the design in a recorded walkthrough or written review.

### Exit artifact

```text
xv6-labs/capstone/
docs/xv6-capstone-design.md
```

## Phase 2 gate

- [ ] Completed xv6 system-call, page-table, traps, COW, network, lock, filesystem, and mmap work.
- [ ] Can trace system calls, faults, scheduling, packets, and writes end to end.
- [ ] Can explain which general-purpose OS abstractions the unikernel will omit.
- [ ] Completed one substantial xv6 capstone.

---

# Phase 3 — Bare-metal x86-64 kernel

## Week 21 — Boot a Rust `no_std` kernel

**Objective:** Reach Rust code on bare-metal x86-64 with a controlled image layout.

### Reading

- [ ] *Writing an OS in Rust*: A Freestanding Rust Binary.
- [ ] *Writing an OS in Rust*: A Minimal Rust Kernel.
- [ ] OSDev: [Boot Sequence](https://wiki.osdev.org/Boot_Sequence).
- [ ] OSDev: [Limine Bare Bones](https://wiki.osdev.org/Limine_Bare_Bones).
- [ ] OSDev: [Higher Half Kernel](https://wiki.osdev.org/Higher_Half_Kernel).

### Implementation

- [ ] Create `oc-kernel` as a `no_std`, `no_main` Rust crate.
- [ ] Boot through Limine.
- [ ] Establish a known stack.
- [ ] Clear `.bss`.
- [ ] Parse the bootloader memory map.
- [ ] Write to COM1 serial.
- [ ] Exit QEMU cleanly.

### Verification

- [ ] Serial output identifies image version and entry address.
- [ ] Memory-map entries are printed and validated.
- [ ] GDB breaks at `_start` and at the Rust entry point.
- [ ] ELF permissions remain RX/R/RW.

### Exit artifact

```text
oc-kernel/boot/
docs/BOOT-ABI-draft.md
```

---

## Week 22 — Kernel testing and diagnostics

**Objective:** Make failures reproducible before adding complexity.

### Reading

- [ ] *Writing an OS in Rust*: Testing.
- [ ] OSDev: [Testing](https://wiki.osdev.org/Testing).
- [ ] OSDev: [Serial Ports](https://wiki.osdev.org/Serial_Ports).
- [ ] OSDev: [Debugging](https://wiki.osdev.org/Debugging).

### Implementation

- [ ] Add a kernel test runner.
- [ ] Add a panic handler with source location where available.
- [ ] Add a QEMU exit device or equivalent machine-readable status.
- [ ] Add GDB command files and symbol loading.
- [ ] Add structured boot milestones.
- [ ] Run kernel tests in CI.

### Verification

- [ ] A passing test exits with success.
- [ ] A deliberate panic exits with failure and emits diagnostics.
- [ ] CI detects timeout and missing output.

### Exit artifact

```text
oc-kernel/tests/
scripts/kernel-test
.gdbinit-oc-kernel
```

---

## Week 23 — GDT, TSS, and CPU state

**Objective:** Establish architectural tables and dedicated fault stacks.

### Reading

- [ ] Intel SDM: segmentation in long mode, descriptor tables, TSS, and IST.
- [ ] OSDev: [Global Descriptor Table](https://wiki.osdev.org/Global_Descriptor_Table).

### Implementation

- [ ] Define and load a GDT.
- [ ] Define a TSS.
- [ ] Allocate a dedicated interrupt stack.
- [ ] Validate segment selectors.
- [ ] Dump relevant CPU state at boot.

### Verification

- [ ] GDT and TSS load successfully.
- [ ] Dedicated stack address is in a valid mapped range.
- [ ] Invalid selector experiments are isolated and documented.

### Exit artifact

```text
oc-kernel/src/arch/x86_64/gdt.rs
oc-kernel/src/arch/x86_64/tss.rs
```

---

## Week 24 — IDT and exception handling

**Objective:** Catch, classify, and report architectural faults.

### Reading

- [ ] *Writing an OS in Rust*: CPU Exceptions.
- [ ] *Writing an OS in Rust*: Double Faults.
- [ ] Intel SDM exception and IDT sections.
- [ ] OSDev: [Interrupt Descriptor Table](https://wiki.osdev.org/Interrupt_Descriptor_Table).
- [ ] OSDev: [Exceptions](https://wiki.osdev.org/Exceptions).

### Implementation

- [ ] Install an IDT.
- [ ] Handle divide error, invalid opcode, general protection, page fault, and double fault.
- [ ] Use the dedicated IST stack for double fault.
- [ ] Print registers, vector, error code, instruction pointer, and fault address.
- [ ] Add deliberate tests for every handler.

### Verification

- [ ] Each deliberate fault reaches the correct handler.
- [ ] A page fault prints the faulting virtual address and decoded error bits.
- [ ] Double-fault handling does not triple fault.

### Exit artifact

```text
oc-kernel/src/arch/x86_64/idt.rs
oc-kernel/tests/exceptions.rs
```

---

## Week 25 — Timers and interrupts

**Objective:** Add an interrupt-driven source of time.

### Reading

- [ ] *Writing an OS in Rust*: Hardware Interrupts.
- [ ] Intel SDM APIC material.
- [ ] OSDev: [APIC](https://wiki.osdev.org/APIC).
- [ ] OSDev: [APIC Timer](https://wiki.osdev.org/APIC_Timer).
- [ ] OSDev: [PIT](https://wiki.osdev.org/Programmable_Interval_Timer).
- [ ] OSDev: [HPET](https://wiki.osdev.org/HPET).

### Implementation

- [ ] Select and document the initial interrupt controller/timer path.
- [ ] Program a periodic or deadline timer.
- [ ] Acknowledge interrupts correctly.
- [ ] Maintain monotonic ticks.
- [ ] Keep the interrupt handler minimal and defer work.

### Verification

- [ ] Tick rate is within measured tolerance.
- [ ] Interrupt count remains stable under load.
- [ ] No interrupt storm after acknowledgement.
- [ ] Timer tests pass under several QEMU CPU settings.

### Exit artifact

```text
oc-kernel/src/time/
oc-kernel/src/arch/x86_64/apic.rs
```

---

## Week 26 — Inspect and design virtual memory

**Objective:** Understand the bootloader-provided mappings and define your own address-space policy.

### Reading

- [ ] *Writing an OS in Rust*: Introduction to Paging.
- [ ] CS:APP Chapter 9 review.
- [ ] Intel SDM paging sections.
- [ ] OSDev: [Page Tables](https://wiki.osdev.org/Page_Tables).
- [ ] OSDev: [Higher Half Kernel](https://wiki.osdev.org/Higher_Half_Kernel).

### Implementation

- [ ] Walk active page tables.
- [ ] Print page size and permission for each kernel region.
- [ ] Define a virtual-address layout document.
- [ ] Define a direct-physical-map strategy.
- [ ] Reserve regions for heap, stacks, MMIO, and temporary mappings.

### Verification

- [ ] The document explains every major virtual range.
- [ ] Page-table walker matches debugger observations.
- [ ] Null page is identified for removal.

### Exit artifact

```text
docs/architecture-decisions/ADR-0002-virtual-address-layout.md
oc-kernel/src/memory/walk.rs
```

---

## Week 27 — Own the mappings and enforce W^X

**Objective:** Create, change, and remove mappings safely.

### Reading

- [ ] *Writing an OS in Rust*: Paging Implementation.
- [ ] Intel SDM page permissions, NX, TLB invalidation, and canonical-address rules.

### Implementation

- [ ] Implement `map`, `unmap`, and permission-update operations.
- [ ] Unmap page zero.
- [ ] Make code RX, read-only data R, and writable data RW+NX.
- [ ] Add a temporary mapping mechanism.
- [ ] Add TLB invalidation where required.
- [ ] Reject overlapping or invalid mappings.

### Verification

- [ ] Write to code triggers a fault.
- [ ] Execute from heap triggers a fault.
- [ ] Access to page zero triggers a fault.
- [ ] Mapping tests include huge-page boundary cases where relevant.

### Exit artifact

```text
oc-kernel/src/memory/paging.rs
oc-kernel/tests/page-permissions.rs
```

---

## Week 28 — Physical frames, heap, stacks, and DMA

**Objective:** Build allocation layers with explicit ownership and telemetry.

### Reading

- [ ] OSTEP Chapter 17 review.
- [ ] *Writing an OS in Rust*: Heap Allocation.
- [ ] *Writing an OS in Rust*: Allocator Designs.
- [ ] OSDev: [Page Frame Allocation](https://wiki.osdev.org/Page_Frame_Allocation).
- [ ] OSDev: [Heap](https://wiki.osdev.org/Heap).

### Implementation

- [ ] Implement an early bump allocator.
- [ ] Implement a physical-frame bitmap or buddy allocator.
- [ ] Implement a heap allocator.
- [ ] Implement stack allocation with guard pages.
- [ ] Implement a page-aligned, physically suitable DMA allocator.
- [ ] Add allocation counters and leak diagnostics.

### Verification

- [ ] Exhaustion returns a controlled error or panic policy.
- [ ] Double-free and invalid-free tests are detected.
- [ ] Guard-page overflow triggers a page fault.
- [ ] DMA buffers satisfy alignment and contiguity requirements you declare.

### Exit artifact

```text
oc-kernel/src/memory/frame_allocator.rs
oc-kernel/src/memory/heap.rs
oc-kernel/src/memory/dma.rs
```

---

## Week 29 — Cooperative execution

**Objective:** Run multiple tasks without process semantics.

### Reading

- [ ] *Writing an OS in Rust*: Async/Await.
- [ ] OSTEP Chapter 33 review.
- [ ] OSDev: [Context Switching](https://wiki.osdev.org/Context_Switching).
- [ ] OSDev: [Scheduling Algorithms](https://wiki.osdev.org/Scheduling_Algorithms).

### Implementation

Choose either stackful cooperative contexts or a `Future`-based executor as the primary model. Document the choice.

- [ ] Implement task creation.
- [ ] Implement a ready queue.
- [ ] Implement wake-up.
- [ ] Implement a sleep/deadline queue.
- [ ] Implement cancellation.
- [ ] Ensure interrupt handlers only enqueue work.

### Verification

- [ ] Multiple tasks make progress.
- [ ] Sleeping tasks do not busy wait.
- [ ] Wake-up cannot be lost.
- [ ] Cancellation releases resources.

### Exit artifact

```text
oc-kernel/src/executor/
docs/architecture-decisions/ADR-0003-execution-model.md
```

---

## Week 30 — Time, entropy, logs, and embedded files

**Objective:** Complete the minimum runtime services needed by higher layers.

### Reading

- [ ] OSDev: [Time and Date](https://wiki.osdev.org/Time_And_Date).
- [ ] OSDev: [Random Number Generator](https://wiki.osdev.org/Random_Number_Generator).
- [ ] OSDev: [Stack Trace](https://wiki.osdev.org/Stack_Trace).

### Implementation

- [ ] Provide monotonic time in a stable unit.
- [ ] Define wall-clock time as externally supplied configuration.
- [ ] Implement entropy initialization and an RNG interface.
- [ ] Implement structured logging with level and subsystem.
- [ ] Add symbolized or partially symbolized backtraces where practical.
- [ ] Embed and inspect a read-only archive.

### Verification

- [ ] Monotonic time never moves backward in normal execution.
- [ ] Entropy failure has an explicit policy.
- [ ] Logs include image/build identity.
- [ ] Embedded files are reproducibly included in the image.

### Exit artifact

```text
oc-kernel/src/time/
oc-kernel/src/entropy/
oc-kernel/src/logging/
oc-kernel/src/archive/
```

## Phase 3 gate

- [ ] Boots through Limine under QEMU.
- [ ] Correct GDT, TSS, IDT, and fault reporting.
- [ ] Timer interrupts and monotonic time.
- [ ] Owned page tables and W^X.
- [ ] Frame, heap, stack, and DMA allocation.
- [ ] Cooperative tasks and wake-ups.
- [ ] Structured logging and automated tests.

---

# Phase 4 — Virtio, networking, storage, and host control

## Week 31 — Device interfaces, MMIO, and DMA contracts

**Objective:** Define driver-facing abstractions before implementing a particular virtio device.

### Reading

- [ ] OSTEP Chapter 36 review.
- [ ] Intel memory-ordering and cacheability material relevant to MMIO/DMA.
- [ ] OSDev: [PCI](https://wiki.osdev.org/PCI).
- [ ] OSDev: [PCI Express](https://wiki.osdev.org/PCI_Express).
- [ ] OSDev: [Virtio](https://wiki.osdev.org/Virtio).

### Implementation

- [ ] Define typed MMIO register access.
- [ ] Define DMA region ownership and address translation.
- [ ] Define `NetDevice`, `BlockDevice`, and generic event-source traits.
- [ ] Build a fake MMIO device for hosted tests.
- [ ] Document reset and failure behaviour.

### Verification

- [ ] Host tests can emulate reads, writes, reset, and interrupts.
- [ ] Driver traits do not expose QEMU-specific types.
- [ ] Every register field has width, access, and state semantics documented.

### Exit artifact

```text
oc-kernel/src/drivers/traits.rs
labs/fake-mmio-device/
```

---

## Week 32 — Split virtqueue core

**Objective:** Implement the transport-independent queue data structure.

### Reading

- [ ] Virtio specification: basic facilities, device status, feature negotiation, and split virtqueues.
- [ ] OSDev: [Virtio](https://wiki.osdev.org/Virtio).

### Implementation

- [ ] Implement descriptor allocation and release.
- [ ] Implement direct and chained descriptors.
- [ ] Implement available and used rings.
- [ ] Handle index wrap-around.
- [ ] Define ownership transitions.
- [ ] Add memory barriers at publication and completion points.
- [ ] Implement queue reset.

### Verification

- [ ] No descriptor is returned twice.
- [ ] Descriptor chains cannot loop indefinitely.
- [ ] Out-of-range descriptor references are rejected.
- [ ] Ring wrap-around tests pass.
- [ ] Broken-ordering tests demonstrate why barriers matter.

### Exit artifact

```text
oc-kernel/src/drivers/virtio/queue.rs
oc-kernel/tests/virtqueue.rs
```

---

## Week 33 — Virtio-MMIO transport

**Objective:** Negotiate and configure a device over the MMIO transport.

### Reading

- [ ] Virtio specification: MMIO transport section.
- [ ] Device initialization and status-state requirements.

### Implementation

- [ ] Discover a virtio-MMIO device.
- [ ] Validate magic, version, device ID, and vendor ID.
- [ ] Negotiate feature bits.
- [ ] Configure queue addresses and size.
- [ ] Drive the status state machine correctly.
- [ ] Handle device failure and reset.
- [ ] Communicate with a toy queue-based device before virtio-net.

### Verification

- [ ] Invalid state transitions are rejected.
- [ ] Unsupported required features fail cleanly.
- [ ] Reset returns the driver to a known state.
- [ ] Toy device transfers a byte buffer in both directions.

### Exit artifact

```text
oc-kernel/src/drivers/virtio/mmio.rs
oc-kernel/tests/virtio-mmio.rs
```

---

## Week 34 — Virtio-net

**Objective:** Send and receive Ethernet frames through a real virtual NIC.

### Reading

- [ ] Virtio specification: network-device section.
- [ ] OSDev: [Network Stack](https://wiki.osdev.org/Network_Stack).

### Implementation

- [ ] Initialize RX and TX queues.
- [ ] Allocate and recycle packet buffers.
- [ ] Start with polling mode.
- [ ] Add interrupt-driven receive after polling is stable.
- [ ] Connect QEMU to a TAP interface.
- [ ] Add packet, byte, drop, and notification counters.

### Verification

- [ ] Raw Ethernet frames leave and enter the guest.
- [ ] Host packet capture matches guest counters.
- [ ] RX buffer ownership remains valid under load.
- [ ] Device reset does not reuse stale buffers.

### Exit artifact

```text
oc-kernel/src/drivers/virtio/net.rs
scripts/setup-tap
```

---

## Week 35 — Ethernet, ARP, IPv4, ICMP, and UDP

**Objective:** Learn packet formats and checksums by implementing a small educational stack.

### Reading

- [ ] OSDev: [Ethernet](https://wiki.osdev.org/Ethernet).
- [ ] OSDev: [ARP](https://wiki.osdev.org/Address_Resolution_Protocol).
- [ ] OSDev: [Internet Protocol](https://wiki.osdev.org/Internet_Protocol).
- [ ] OSDev: [ICMP](https://wiki.osdev.org/Internet_Control_Message_Protocol).
- [ ] OSDev: [UDP](https://wiki.osdev.org/User_Datagram_Protocol).
- [ ] Relevant RFCs.

### Implementation

- [ ] Parse and generate Ethernet II frames.
- [ ] Implement ARP request/reply and a bounded cache.
- [ ] Validate IPv4 version, header length, total length, and checksum.
- [ ] Implement ICMP echo request/reply.
- [ ] Implement UDP parsing, checksum policy, and echo service.
- [ ] Fuzz each parser in hosted mode.

### Verification

- [ ] Host can ping the guest.
- [ ] Host can use UDP echo.
- [ ] Truncated and malformed packets are rejected safely.
- [ ] ARP cache has bounded memory and expiry behaviour.

### Exit artifact

```text
labs/network-protocol-lab/
oc-kernel/src/net/educational/
fuzz/network-parsers/
```

---

## Week 36 — Integrate smoltcp and serve HTTP

**Objective:** Replace the educational stack with a maintained product stack.

### Reading

- [ ] smoltcp documentation, examples, and device traits.
- [ ] OSTEP event-driven concurrency review.

### Implementation

- [ ] Adapt virtio-net to smoltcp device tokens.
- [ ] Configure an IPv4 address statically; add DHCP later if useful.
- [ ] Implement TCP listeners.
- [ ] Implement an HTTP service with:
  - [ ] `GET /health`;
  - [ ] `GET /version`;
  - [ ] `GET /metrics`;
  - [ ] `POST /echo`.
- [ ] Add bounded request size, connection count, and timeouts.

### Verification

- [ ] Sustained TCP requests succeed.
- [ ] Idle and slow clients time out.
- [ ] Memory remains bounded under load.
- [ ] Metrics expose queue, packet, connection, and allocation state.

### Exit artifact

```text
oc-kernel/src/net/smoltcp_adapter.rs
apps/http-service/
```

---

## Week 37 — Virtio-block and simple durable state

**Objective:** Learn block request semantics without implementing a large filesystem.

### Reading

- [ ] Virtio specification: block-device section.
- [ ] OSTEP Chapters 40 and 42 review.
- [ ] OSDev storage/filesystem material as needed.

### Implementation

- [ ] Implement block reads.
- [ ] Implement block writes.
- [ ] Implement flush.
- [ ] Handle read-only devices.
- [ ] Add either:
  - [ ] a read-only archive over block storage; or
  - [ ] an append-only key-value log.
- [ ] Define crash and partial-failure semantics.

### Verification

- [ ] Reads and writes survive a guest restart where expected.
- [ ] Flush is observable in a controlled test.
- [ ] Read-only attachment rejects writes.
- [ ] Out-of-range requests fail safely.

### Exit artifact

```text
oc-kernel/src/drivers/virtio/block.rs
apps/kv-service/
```

---

## Week 38 — Virtio-vsock and host control

**Objective:** Add a narrow control path independent of the guest network interface.

### Reading

- [ ] Virtio specification: socket-device section.
- [ ] Firecracker [vsock documentation](https://github.com/firecracker-microvm/firecracker/blob/main/docs/vsock.md).

### Implementation

- [ ] Implement or integrate virtio-vsock.
- [ ] Define a versioned control protocol.
- [ ] Support configuration delivery.
- [ ] Support readiness notification.
- [ ] Support structured logs or log-control messages.
- [ ] Support quiesce and shutdown requests.
- [ ] Bound message size and connection count.

### Verification

- [ ] Host can deliver configuration before application readiness.
- [ ] Guest sends a machine-readable ready event.
- [ ] Invalid control messages do not crash the guest.
- [ ] Shutdown completes within a bounded timeout.

### Exit artifact

```text
oc-kernel/src/drivers/virtio/vsock.rs
docs/HOST-CONTROL-PROTOCOL.md
```

## Phase 4 gate

- [ ] Generic split virtqueue.
- [ ] Virtio-MMIO transport.
- [ ] Virtio-net with TAP integration.
- [ ] Educational UDP stack and fuzzed parsers.
- [ ] smoltcp TCP/HTTP service.
- [ ] Virtio-block with persistence tests.
- [ ] Virtio-vsock control protocol.

---

# Phase 5 — Virtualization, KVM, and a small VMM

## Week 39 — Virtualization models

**Objective:** Understand the major ways systems virtualize privileged execution.

### Reading

- [ ] Popek and Goldberg.
- [ ] *Xen and the Art of Virtualization*.
- [ ] *KVM: the Linux Virtual Machine Monitor*.
- [ ] Optional: *Virtual Machines*, Chapters 1 and 8.

### Implementation

- [ ] Write a comparison of trap-and-emulate, binary translation, paravirtualization, and hardware-assisted virtualization.
- [ ] Draw the protection boundaries in Xen, KVM/QEMU, and Firecracker.
- [ ] Identify where device emulation runs in each design.

### Verification

- [ ] You can explain why classic x86 was difficult to virtualize.
- [ ] You can distinguish guest virtual, guest physical, and host virtual addresses.
- [ ] You can explain why virtio is paravirtualized even when the CPU is hardware-virtualized.

### Exit artifact

```text
docs/notes/week-39-virtualization-models.md
```

---

## Week 40 — VMX, SVM, EPT, and NPT

**Objective:** Understand the hardware mechanisms KVM exposes without implementing them directly.

### Reading

- [ ] Intel SDM VMX overview, VM entry/exit, VMCS, and EPT sections.
- [ ] AMD architecture material on SVM, VMCB, and NPT.

### Implementation

- [ ] Draw VMX root/non-root transitions.
- [ ] Draw a VM-exit handling sequence.
- [ ] Compare VMCS and VMCB.
- [ ] Compare EPT and NPT.
- [ ] Map common exit reasons to VMM responsibilities.

### Verification

- [ ] You can explain why a guest page fault differs from an EPT/NPT violation.
- [ ] You can explain CPUID filtering, MSR handling, and interrupt injection conceptually.
- [ ] You can identify which work KVM performs in the kernel and which remains in userspace.

### Exit artifact

```text
docs/notes/week-40-hardware-virtualization.md
```

---

## Week 41 — Create and run a KVM VM

**Objective:** Use the raw KVM API to execute a tiny guest.

### Reading

- [ ] KVM API: general description.
- [ ] KVM, VM, and vCPU file descriptors.
- [ ] `KVM_GET_API_VERSION`, `KVM_CREATE_VM`, memory-region APIs, `KVM_CREATE_VCPU`, and `KVM_RUN`.

### Implementation

- [ ] Open `/dev/kvm`.
- [ ] Verify API version and required capabilities.
- [ ] Create a VM.
- [ ] Allocate and register guest memory.
- [ ] Create a vCPU.
- [ ] Map `kvm_run`.
- [ ] Run a tiny real-mode guest that performs port I/O and halts.
- [ ] Handle I/O, HLT, shutdown, and unexpected exits.

### Verification

- [ ] Guest output is visible through an I/O exit.
- [ ] Every exit reason is logged.
- [ ] Unexpected exits produce enough state to debug.

### Exit artifact

```text
oc-vmm/src/kvm.rs
oc-vmm/examples/real-mode-hello.rs
```

---

## Week 42 — ELF loading and long-mode entry

**Objective:** Boot your own x86-64 image without QEMU.

### Reading

- [ ] KVM x86 register and special-register APIs.
- [ ] Review ELF loading and page-table setup.

### Implementation

- [ ] Parse your kernel ELF.
- [ ] Copy loadable segments into guest memory.
- [ ] Zero segment memory beyond file size.
- [ ] Create initial page tables.
- [ ] Initialize control registers and long-mode state.
- [ ] Set RIP, RSP, and boot-information pointer.
- [ ] Capture serial-like output.

### Verification

- [ ] `oc-vmm` reaches the same kernel entry as QEMU.
- [ ] Segment permissions and addresses match ELF metadata.
- [ ] Invalid ELF images are rejected before vCPU execution.

### Exit artifact

```text
oc-vmm/src/elf_loader.rs
oc-vmm/src/x86_64/boot.rs
```

---

## Week 43 — VMM event loop and interrupt injection

**Objective:** Build the host-side concurrency model for vCPUs and devices.

### Reading

- [ ] KVM interrupt, irqchip, eventfd, and vCPU-state APIs required by your design.
- [ ] `epoll` and `eventfd` documentation.

### Implementation

- [ ] Run the vCPU in a dedicated thread.
- [ ] Build an `epoll`-based device and control event loop.
- [ ] Use `eventfd` for notifications.
- [ ] Add graceful stop and shutdown.
- [ ] Inject one synthetic interrupt.
- [ ] Record exit count and handling latency by reason.

### Verification

- [ ] Interrupt reaches a guest handler.
- [ ] Stop request reliably terminates the vCPU loop.
- [ ] VM-exit report is generated after every run.

### Exit artifact

```text
oc-vmm/src/vcpu.rs
oc-vmm/src/event_loop.rs
benches/vm-exits/
```

---

## Week 44 — Device model

**Objective:** Understand the VMM side of MMIO and device state.

### Reading

- [ ] Review KVM MMIO exits.
- [ ] Review virtqueue ownership and notification.

### Implementation

First implement a simple MMIO register device:

```text
offset 0x00: device ID
offset 0x04: status
offset 0x08: input
offset 0x0c: output
```

Then:

- [ ] validate access width and offset;
- [ ] inject an interrupt;
- [ ] serialize the device state;
- [ ] implement a small descriptor-based queue device;
- [ ] fuzz malformed guest requests.

### Verification

- [ ] Invalid accesses do not corrupt VMM memory.
- [ ] Device reset returns a defined state.
- [ ] Serialized state restores exactly.
- [ ] Guest and VMM agree on ownership transitions.

### Exit artifact

```text
oc-vmm/src/devices/toy_mmio.rs
oc-vmm/src/devices/toy_queue.rs
```

---

## Week 45 — Stopped-state snapshots

**Objective:** Save and restore a quiesced VM with explicit compatibility metadata.

### Reading

- [ ] KVM get/set registers and events.
- [ ] Firecracker snapshot documentation.

### Implementation

Define a snapshot containing:

```text
format version
image digest
configuration digest
guest memory
vCPU general registers
vCPU special registers
interrupt state
device state
clock state
```

- [ ] Implement stop-the-world capture.
- [ ] Implement restore into a new VMM process.
- [ ] Validate image and configuration identity.
- [ ] Reject unsupported snapshot versions.

### Verification

- [ ] Restored guest continues from the expected point.
- [ ] Timers do not jump backward.
- [ ] Wrong image digest is rejected.
- [ ] Device state matches pre-snapshot state.

### Exit artifact

```text
oc-vmm/src/snapshot/
docs/SNAPSHOT-FORMAT-draft.md
```

---

## Week 46 — Refactor with rust-vmm components

**Objective:** Compare first-principles understanding with reusable production-oriented components.

### Reading

- [ ] rust-vmm community and selected crates.
- [ ] Documentation for `kvm-ioctls`, `vm-memory`, and any virtio crates you use.

### Implementation

- [ ] Replace selected raw ioctl wrappers with `kvm-ioctls`.
- [ ] Replace guest-memory boilerplate with `vm-memory` where useful.
- [ ] Preserve your own architecture boundaries.
- [ ] Benchmark code size, complexity, and runtime behaviour before and after.

### Verification

- [ ] All previous VMM tests still pass.
- [ ] The refactor reduces boilerplate without obscuring lifecycle invariants.
- [ ] The report identifies which components you would use in production.

### Exit artifact

```text
docs/notes/week-46-rust-vmm-evaluation.md
```

## Phase 5 gate

- [ ] Raw KVM VM and vCPU creation.
- [ ] Real-mode guest execution.
- [ ] x86-64 ELF loading and long-mode entry.
- [ ] Event loop and interrupt injection.
- [ ] Toy MMIO and queue devices.
- [ ] Stopped-state save and restore.
- [ ] rust-vmm evaluation.

---

# Phase 6 — Library OS and unikernel systems archaeology

For every paper, use the [paper-review template](#13-paper-review-template).

## Week 47 — Exokernel and Drawbridge

**Objective:** Compare bottom-up resource exposure with top-down compatibility.

### Reading

- [ ] *Exokernel: An Operating System Architecture for Application-Level Resource Management*.
- [ ] *Rethinking the Library OS from the Top Down*.

### Practical work

- [ ] Write separate paper reviews.
- [ ] Draw each system’s protection boundary.
- [ ] List the abstractions moved into the library OS.
- [ ] Design a hypothetical minimal host ABI with memory, execution, time, and I/O-stream operations.
- [ ] Compare that ABI with your current platform traits.

### Verification

- [ ] You can explain protection versus resource-management policy.
- [ ] You can explain why compatibility changes the direction of library-OS design.
- [ ] Your host ABI has explicit versioning and failure semantics.

### Exit artifact

```text
docs/paper-reviews/exokernel.md
docs/paper-reviews/drawbridge.md
docs/architecture-decisions/ADR-0004-host-abi.md
```

---

## Week 48 — MirageOS

**Objective:** Study compile-time specialization and typed library composition.

### Reading

- [ ] *Unikernels: Library Operating Systems for the Cloud*.
- [ ] *Rise of the Virtual Library Operating System*.
- [ ] MirageOS documentation required to build a small service.

### Practical work

- [ ] Build a MirageOS HTTP service.
- [ ] Run a hosted/process target where supported.
- [ ] Run a virtualized target.
- [ ] Inspect the dependency graph and resulting image.
- [ ] Record configuration, boot, networking, and debugging workflow.

### Verification

- [ ] You can explain when specialization occurs.
- [ ] You can identify what replaces a conventional distribution and init system.
- [ ] You have measured image size and startup time.

### Exit artifact

```text
comparisons/mirage-http/
docs/paper-reviews/mirageos.md
```

---

## Week 49 — Solo5

**Objective:** Study a narrow execution substrate between library OS and host.

### Reading

- [ ] Solo5 repository documentation and architecture material.
- [ ] Solo5 manifest, ABI, and tender/binding documentation.

### Practical work

- [ ] Draw the split among application, library OS, bindings, and tender.
- [ ] Inspect ABI versioning and application manifest design.
- [ ] Run a hardware-virtualized and process-style target where practical.
- [ ] Compare its host interface with your platform abstraction.

### Verification

- [ ] You can explain why a narrow substrate improves portability.
- [ ] You can identify which responsibilities belong in `oc-uk` and which belong in a VMM/tender.
- [ ] You have proposed an embedded manifest format for `oc-uk`.

### Exit artifact

```text
docs/paper-reviews/solo5.md
docs/architecture-decisions/ADR-0005-image-manifest.md
```

---

## Week 50 — ClickOS and Jitsu

**Objective:** Study network-specialized images and request-driven activation.

### Reading

- [ ] *ClickOS and the Art of Network Function Virtualization*.
- [ ] *Jitsu: Just-In-Time Summoning of Unikernels*.

### Practical work

- [ ] Map the boot path and network attachment model in both systems.
- [ ] Identify control-plane responsibilities.
- [ ] Design an Opencomputer request-to-instance sequence.
- [ ] Define readiness and termination events.
- [ ] Estimate which startup milestones dominate latency in your current kernel.

### Verification

- [ ] Sequence diagram covers request, image selection, VM creation, network setup, readiness, and teardown.
- [ ] You can distinguish cold boot, restored boot, and already-running routing.
- [ ] You have identified metrics needed for startup optimization.

### Exit artifact

```text
docs/paper-reviews/clickos.md
docs/paper-reviews/jitsu.md
docs/opencomputer-startup-sequence.md
```

---

## Week 51 — Unikraft

**Objective:** Study configurable micro-library composition and application porting.

### Reading

- [ ] *Unikraft: Fast, Specialized Unikernels the Easy Way*.
- [ ] Unikraft documentation for configuration, libraries, and application porting.

### Practical work

- [ ] Build hello world.
- [ ] Build one existing application, such as a web server, Redis, or SQLite example.
- [ ] Inspect allocator, scheduler, libc, syscall, and network choices.
- [ ] Record application-porting friction.
- [ ] Compare native source porting with binary compatibility.

### Verification

- [ ] You can explain Unikraft’s library categories and configuration model.
- [ ] You have a measured comparison against `oc-kernel` or MirageOS.
- [ ] You have an explicit position on native API versus POSIX support.

### Exit artifact

```text
comparisons/unikraft/
docs/paper-reviews/unikraft.md
```

---

## Week 52 — Alternative targets, large-scale composition, and security

**Objective:** Finalize the architecture of `oc-uk` before implementation.

### Reading

- [ ] *Unikernels as Processes*.
- [ ] *Programming Unikernels in the Large*.
- [ ] *A Security Perspective on Unikernels*.
- [ ] *Assessing Unikernel Security*.

### Practical work

- [ ] Complete the comparison matrix below.
- [ ] Write the `oc-uk` architecture decision record.
- [ ] Draft the threat model.
- [ ] Define hosted Linux, QEMU, `oc-vmm`, and Firecracker targets.
- [ ] Define native API and limited POSIX strategy.

### Required comparison matrix

| Dimension | MirageOS | Solo5 | Unikraft | `oc-uk` |
|---|---|---|---|---|
| Application languages | | | | |
| Application ABI | | | | |
| Host/platform ABI | | | | |
| Build specialization | | | | |
| Scheduling | | | | |
| Memory management | | | | |
| Networking | | | | |
| Block storage | | | | |
| libc/POSIX support | | | | |
| VM/process targets | | | | |
| Configuration | | | | |
| Debugging | | | | |
| Snapshot model | | | | |
| Isolation boundary | | | | |

### Verification

The architecture decision record justifies:

- [ ] one address space;
- [ ] static application linkage;
- [ ] native API first;
- [ ] limited POSIX later;
- [ ] cooperative execution first;
- [ ] virtio devices;
- [ ] platform-independent driver interfaces;
- [ ] hosted, QEMU, `oc-vmm`, and Firecracker backends;
- [ ] narrow host-control ABI;
- [ ] content-addressed ELF artifact.

### Exit artifact

```text
docs/architecture-decisions/ADR-0006-oc-uk-architecture.md
docs/THREAT-MODEL-draft.md
docs/unikernel-comparison.md
```

## Phase 6 gate

- [ ] Reviewed the foundational library-OS and unikernel papers.
- [ ] Built at least one MirageOS and one Unikraft application.
- [ ] Studied Solo5’s ABI/substrate split.
- [ ] Completed architecture and threat-model drafts.
- [ ] Can defend `oc-uk`’s application and platform interfaces.

---

# Phase 7 — Build and deploy `oc-uk`

## Week 53 — Project structure, boot ABI, and platform abstraction

**Objective:** Create the final library-OS repository and normalized platform contract.

### Reading

- [ ] Revisit Solo5 manifest and ABI material.
- [ ] Revisit MirageOS composition material.
- [ ] Review ELF note format.

### Implementation

Create this structure:

```text
oc-uk/
├── arch/x86_64/
├── kernel/boot/
├── kernel/memory/
├── kernel/interrupt/
├── kernel/time/
├── kernel/executor/
├── kernel/entropy/
├── platform/hosted/
├── platform/qemu/
├── platform/firecracker/
├── drivers/virtio/
├── libos/io/
├── libos/net/
├── libos/fs/
├── libos/sync/
├── libos/task/
├── libc-shim/
├── sdk/
├── apps/
├── tests/
├── fuzz/
└── benches/
```

- [ ] Define `BootInfo`.
- [ ] Define a `Platform` trait.
- [ ] Define normalized device descriptors.
- [ ] Embed a versioned manifest as an ELF note.
- [ ] Implement a hosted Linux backend for unit/integration tests.
- [ ] Define ABI compatibility rules.

Suggested platform interface:

```rust
pub trait Platform {
    fn memory_regions(&self) -> &[MemoryRegion];
    fn monotonic_time(&self) -> core::time::Duration;
    fn fill_entropy(&self, output: &mut [u8]) -> Result<(), PlatformError>;
    fn devices(&self) -> &[DeviceDescriptor];
    fn shutdown(&self, reason: ShutdownReason) -> !;
}
```

### Verification

- [ ] Core library-OS tests run as ordinary Linux processes.
- [ ] Manifest parser rejects incompatible versions.
- [ ] QEMU-specific types do not leak into application-facing APIs.
- [ ] ABI documents define ownership, layout, and lifetime.

### Exit artifact

```text
oc-uk/
docs/BOOT-ABI.md
docs/PLATFORM-ABI.md
```

---

## Week 54 — Refactor runtime services

**Objective:** Move proven kernel mechanisms into clean, testable library-OS components.

### Reading

- [ ] Revisit your allocator, executor, time, entropy, and logging notes.
- [ ] Review unsafe Rust and data-layout rules for all public low-level APIs.

### Implementation

- [ ] Refactor physical-memory and mapping code.
- [ ] Refactor heap, stack, and DMA allocation.
- [ ] Refactor executor, wait queues, and timers.
- [ ] Refactor entropy and RNG.
- [ ] Refactor logging and panic diagnostics.
- [ ] Define initialization ordering.
- [ ] Make read-only configuration immutable after initialization.

### Verification

- [ ] Hosted tests cover most platform-independent logic.
- [ ] Bare-metal QEMU tests cover architecture-specific paths.
- [ ] Every unsafe module contains invariants and safety contracts.
- [ ] W^X and guard-page tests continue to pass.

### Exit artifact

```text
oc-uk/kernel/
oc-uk/tests/runtime/
```

---

## Week 55 — Production HTTP unikernel

**Objective:** Produce the first useful, bounded application-specific image.

### Reading

- [ ] Revisit virtio-net and smoltcp integration.
- [ ] Review HTTP parser/library security assumptions.

### Implementation

- [ ] Integrate virtio-net and smoltcp into `oc-uk`.
- [ ] Implement a native `UnikernelApp` interface.
- [ ] Build `http-service` with health, version, metrics, and echo endpoints.
- [ ] Add bounded queues, request size, connections, and timeouts.
- [ ] Add readiness and graceful shutdown.
- [ ] Emit build, image, and configuration digests in logs.

Suggested application interface:

```rust
pub trait UnikernelApp {
    fn init(config: &AppConfig) -> Result<Self, AppError>
    where
        Self: Sized;

    fn run(&mut self, runtime: &Runtime) -> Result<(), AppError>;

    fn quiesce(&mut self, reason: QuiesceReason) -> Result<(), AppError>;

    fn shutdown(&mut self, reason: ShutdownReason);
}
```

### Verification

- [ ] Image boots and becomes ready under QEMU.
- [ ] Sustained load does not cause unbounded memory growth.
- [ ] Slow-client and malformed-request tests pass.
- [ ] Application shuts down cleanly through host control.

### Exit artifact

```text
oc-uk/apps/http-service/
docs/APPLICATION-ABI.md
```

---

## Week 56 — Storage, configuration, secrets, and quiescing

**Objective:** Add durable data and a safe checkpoint boundary.

### Reading

- [ ] Revisit virtio-block and crash-consistency material.
- [ ] Review Firecracker snapshot guidance.

### Implementation

- [ ] Integrate virtio-block.
- [ ] Attach a separate data device.
- [ ] Implement an append-only key-value service or other bounded durable store.
- [ ] Deliver configuration over vsock.
- [ ] Inject secrets without embedding them in the image.
- [ ] Implement application quiesce.
- [ ] Flush block operations before reporting quiesced state.

### Verification

- [ ] Data persists across clean restart.
- [ ] Snapshot is taken only after quiesce acknowledgement.
- [ ] Secrets are absent from the immutable image and routine logs.
- [ ] Wrong data-volume identity is rejected on restore.

### Exit artifact

```text
oc-uk/apps/kv-service/
docs/QUIESCE-PROTOCOL.md
```

---

## Week 57 — Stable C ABI and limited compatibility layer

**Objective:** Port one nontrivial C application without claiming complete POSIX support.

### Reading

- [ ] System V ABI and C FFI layout review.
- [ ] Only the Unix/POSIX interfaces required by your selected application.
- [ ] Unikraft porting material.

### Implementation

- [ ] Define a stable C entry point.
- [ ] Define opaque handles rather than exposing Rust layouts.
- [ ] Implement the minimal required calls, potentially including:
  - [ ] `read`;
  - [ ] `write`;
  - [ ] `close`;
  - [ ] `clock_gettime`;
  - [ ] `nanosleep`;
  - [ ] `getrandom`;
  - [ ] `mmap`/`munmap`;
  - [ ] `poll`;
  - [ ] socket calls.
- [ ] Port SQLite with a custom VFS or a small C HTTP server.
- [ ] Create an explicit compatibility ledger.

Suggested C entry:

```c
struct uk_boot_info;
int uk_main(const struct uk_boot_info *boot_info);
```

### Verification

- [ ] C application runs under QEMU.
- [ ] Unsupported operations fail predictably.
- [ ] ABI version mismatch is rejected.
- [ ] Compatibility ledger distinguishes implemented, partial, unsupported, and semantically different calls.

### Exit artifact

```text
oc-uk/libc-shim/
oc-uk/apps/c-port/
docs/POSIX-COMPATIBILITY.md
```

---

## Week 58 — Firecracker target

**Objective:** Boot the same application image under Firecracker.

### Reading

- [ ] Firecracker [design](https://github.com/firecracker-microvm/firecracker/blob/main/docs/design.md).
- [ ] Firecracker [kernel/rootfs setup](https://github.com/firecracker-microvm/firecracker/blob/main/docs/rootfs-and-kernel-setup.md).
- [ ] Firecracker [vsock documentation](https://github.com/firecracker-microvm/firecracker/blob/main/docs/vsock.md).

### Implementation

- [ ] Implement the Firecracker-compatible boot path.
- [ ] Normalize memory map and command line into `BootInfo`.
- [ ] Discover/configure virtio-MMIO devices.
- [ ] Validate virtio-net.
- [ ] Validate virtio-block.
- [ ] Validate vsock.
- [ ] Implement Firecracker-compatible shutdown signalling.
- [ ] Add a Firecracker integration-test harness.

### Verification

- [ ] Same HTTP application works under QEMU and Firecracker.
- [ ] Same data format works on the attached block device.
- [ ] Readiness and shutdown work through vsock.
- [ ] Platform-specific code is confined to the Firecracker backend.

### Exit artifact

```text
oc-uk/platform/firecracker/
tests/firecracker/
```

---

## Week 59 — Security, fuzzing, failure injection, and snapshot safety

**Objective:** Treat the unikernel and snapshots as hostile-input surfaces rather than assuming small means secure.

### Reading

- [ ] *A Security Perspective on Unikernels*.
- [ ] *Assessing Unikernel Security*.
- [ ] Firecracker design and snapshot-security guidance.

### Implementation

- [ ] Enforce W^X.
- [ ] Keep null page unmapped.
- [ ] Add stack guard pages and stack protection where supported.
- [ ] Bound all packet, queue, control, and application parsers.
- [ ] Fuzz ELF, manifest, network, virtqueue, control, block-metadata, and snapshot parsers.
- [ ] Add device allowlisting.
- [ ] Add snapshot authentication and encryption at the platform layer.
- [ ] Reseed entropy after restore or clone.
- [ ] Rotate ephemeral credentials after restore where required.
- [ ] Inject packet loss, malformed descriptors, delayed interrupts, block errors, OOM, and control-channel failures.

Quiesce sequence:

```text
1. Stop accepting new work.
2. Drain or cancel in-flight application operations.
3. Flush block writes.
4. Drain or disable device queues.
5. Record application checkpoint metadata.
6. Report quiesced state.
7. Capture vCPU, memory, and VMM device state.
8. Resume or terminate.
```

Restore sequence:

```text
1. Validate snapshot format, image, configuration, CPU, and device topology.
2. Restore memory and device state.
3. Correct monotonic-time accounting.
4. Reseed entropy.
5. Rotate ephemeral credentials if required.
6. Re-establish host control.
7. Validate attached block identities.
8. Resume application work.
```

### Verification

- [ ] Fuzz targets run continuously or on every relevant change.
- [ ] Restored clones do not share RNG state.
- [ ] Wrong image/configuration/device topology is rejected.
- [ ] Threat model covers compromised guest and malicious virtual-device responses.

### Exit artifact

```text
docs/THREAT-MODEL.md
docs/SNAPSHOT-FORMAT.md
fuzz/
tests/failure-injection/
```

---

## Week 60 — Opencomputer integration and production capstone

**Objective:** Turn the research implementation into a reproducible runtime artifact and operational workflow.

### Reading

- [ ] Review your own architecture, ABI, snapshot, and threat-model documents.
- [ ] Review Nix closure and reproducible-build techniques required by the build pipeline.

### Implementation

Build this pipeline:

```text
application source
        │
        ▼
Nix-pinned compiler and dependencies
        │
        ▼
ukbuild component selection
        │
        ▼
static link
        │
        ▼
ELF + embedded manifest
        │
        ▼
validation and signing
        │
        ▼
content-addressed artifact store
```

- [ ] Implement `ukbuild` or an equivalent build wrapper.
- [ ] Validate architecture, ABI version, device requirements, and ELF permissions.
- [ ] Calculate image and configuration digests.
- [ ] Produce a signed artifact manifest.
- [ ] Implement an Opencomputer runtime adapter.
- [ ] Allocate TAP interfaces and vsock CIDs.
- [ ] Attach block devices and snapshot identities.
- [ ] Capture logs, readiness, metrics, and termination reason.
- [ ] Run the production capstone under load and failure injection.
- [ ] Complete the benchmark matrix.

Suggested artifact manifest:

```json
{
  "media_type": "application/vnd.opencomputer.unikernel.v1",
  "digest": "sha256:...",
  "architecture": "x86_64",
  "platforms": ["qemu-kvm", "firecracker"],
  "minimum_memory_bytes": 67108864,
  "vcpus": 1,
  "devices": ["net", "vsock", "block"],
  "application_abi": 1,
  "platform_abi": 1,
  "config_digest": "sha256:...",
  "signature": "..."
}
```

Suggested runtime interface:

```go
type UnikernelSpec struct {
    ImageDigest  string
    Platform     string
    VCPUs        int
    MemoryBytes  uint64
    KernelArgs   []string
    Network      NetworkSpec
    BlockDevices []BlockSpec
    VsockCID     uint32
    Secrets      []SecretRef
}

type UnikernelRuntime interface {
    Create(ctx context.Context, spec UnikernelSpec) (Instance, error)
    Start(ctx context.Context, id string) error
    Stop(ctx context.Context, id string) error
    Snapshot(ctx context.Context, id string) (Snapshot, error)
    Restore(ctx context.Context, snapshot Snapshot) (Instance, error)
    Delete(ctx context.Context, id string) error
}
```

### Verification

- [ ] Clean build is reproducible from a fresh checkout.
- [ ] Artifact identity is content-addressed and signed.
- [ ] QEMU and Firecracker runtimes launch the same logical application.
- [ ] Block snapshot and VM runtime snapshot remain distinct objects.
- [ ] Cold start, restore, load, shutdown, and fault tests pass.
- [ ] Documentation is sufficient for another engineer to build and operate the system.

### Exit artifact

```text
tools/ukbuild/
opencomputer/runtime/unikernel/
docs/ARCHITECTURE.md
docs/OPERATIONS.md
docs/PERFORMANCE.md
docs/PORTING.md
```

## Phase 7 gate

- [ ] One Rust application and one C application.
- [ ] QEMU, `oc-vmm`, and Firecracker targets.
- [ ] Virtio-net, block, and vsock.
- [ ] Stable application and platform ABI documents.
- [ ] Threat model and fuzzing.
- [ ] Quiesced snapshot/restore path.
- [ ] Nix-pinned reproducible artifact pipeline.
- [ ] Opencomputer runtime adapter.
- [ ] Final benchmark and operations documentation.

---

# 9. Testing requirements

Every subsystem should have four forms of verification:

| Test class | Example |
|---|---|
| Hosted unit test | Parse an ELF or packet in an ordinary Linux process |
| VM integration test | Boot QEMU and observe a machine-readable result |
| Negative test | Supply malformed descriptors, packets, or page mappings |
| Benchmark | Measure cycles, allocations, VM exits, or elapsed startup time |

## 9.1 Memory tests

- [ ] Allocate every available frame.
- [ ] Detect double free.
- [ ] Reject unaligned mapping.
- [ ] Reject mapping over protected kernel regions.
- [ ] Verify non-executable heap.
- [ ] Verify non-writable code.
- [ ] Verify unmapped null page.
- [ ] Force out-of-memory.
- [ ] Trigger stack guard page.

## 9.2 Exception and interrupt tests

- [ ] Divide error.
- [ ] Invalid opcode.
- [ ] General-protection fault.
- [ ] Page fault with decoded cause.
- [ ] Nested fault.
- [ ] Double fault.
- [ ] Timer storm.
- [ ] Unexpected vector.

## 9.3 Virtio tests

- [ ] Invalid feature negotiation.
- [ ] Descriptor loop.
- [ ] Out-of-range descriptor.
- [ ] Duplicate completion.
- [ ] Used-index wrap-around.
- [ ] Device reset during request.
- [ ] Notification before memory publication.
- [ ] Stale completion after reset.

## 9.4 Networking tests

- [ ] Truncated Ethernet frame.
- [ ] Invalid IPv4 header length.
- [ ] Invalid checksum.
- [ ] Oversized UDP payload.
- [ ] ARP cache exhaustion.
- [ ] TCP timeout.
- [ ] Packet loss.
- [ ] Packet duplication.
- [ ] Slow HTTP client.
- [ ] Oversized HTTP request.

## 9.5 Block and snapshot tests

- [ ] Out-of-range block operation.
- [ ] Read-only device write.
- [ ] Block read error.
- [ ] Block write error.
- [ ] Snapshot with active timer.
- [ ] Snapshot with queued packets.
- [ ] Snapshot during block write.
- [ ] Restore with wrong image digest.
- [ ] Restore with wrong data volume.
- [ ] Restore with incompatible CPU configuration.
- [ ] Clone snapshot twice and verify entropy divergence.

---

# 10. Scope control

## Do not implement before the first useful HTTP unikernel

- custom firmware or a custom bootloader;
- shell;
- users and groups;
- `fork`;
- `exec`;
- dynamic linking;
- arbitrary shared libraries;
- complete Unix signals;
- complete POSIX;
- ext4;
- USB;
- GPU support;
- physical NIC drivers;
- multiple production CPU architectures;
- production TCP from scratch;
- SMP before the single-vCPU system is stable;
- live migration.

## Implement first

```text
Limine boot
x86-64
one vCPU
one address space
static application
cooperative executor
virtio-MMIO
virtio-net
smoltcp
virtio-block
read-only embedded archive
virtio-vsock
native application API
small stable C ABI
QEMU and Firecracker
```

The handwritten Ethernet/ARP/IPv4/ICMP/UDP stack is a learning exercise. Use a maintained TCP/IP stack for the product unless networking itself becomes the research focus.

---

# 11. Graduation criteria

## 11.1 Kernel and runtime

- [ ] Boots under QEMU/KVM.
- [ ] Boots under Firecracker.
- [ ] Boots under `oc-vmm`.
- [ ] Uses one address space.
- [ ] Supports one statically linked application.
- [ ] Provides frame, heap, stack, and DMA allocation.
- [ ] Provides timers, entropy, and structured logs.
- [ ] Provides cooperative tasks and event waiting.
- [ ] Has no writable-executable mapping.
- [ ] Uses stack guard pages.
- [ ] Keeps page zero unmapped.

## 11.2 Devices

- [ ] Virtio-net.
- [ ] Virtio-block.
- [ ] Virtio-vsock.
- [ ] Queue reset.
- [ ] Malformed-input handling.
- [ ] Read-only and writable block attachments.

## 11.3 Applications and APIs

- [ ] Native Rust HTTP service.
- [ ] Native Rust persistent key-value service.
- [ ] One nontrivial C application.
- [ ] Versioned application ABI.
- [ ] Versioned platform ABI.
- [ ] Explicit POSIX compatibility ledger.

## 11.4 Hypervisor knowledge

- [ ] Raw KVM VM and vCPU setup.
- [ ] Guest-memory registration.
- [ ] x86-64 ELF loading.
- [ ] Long-mode setup.
- [ ] MMIO exit handling.
- [ ] Interrupt injection.
- [ ] Device-state serialization.
- [ ] Stopped-state restore.

## 11.5 Security

- [ ] Threat model.
- [ ] W^X.
- [ ] Stack guards and canaries where supported.
- [ ] Fuzzed ELF, packet, virtqueue, control, and snapshot parsers.
- [ ] Entropy reseeding after restore and cloning.
- [ ] Authenticated and encrypted snapshot artifacts.
- [ ] Device allowlist.
- [ ] Bounded network queues and application requests.
- [ ] Host assumes a compromised guest may behave maliciously.

## 11.6 Opencomputer integration

A deployment should be representable as:

```json
{
  "image_digest": "sha256:...",
  "architecture": "x86_64",
  "platform": "firecracker",
  "memory_bytes": 67108864,
  "vcpus": 1,
  "devices": ["net", "vsock", "block"],
  "config_digest": "sha256:...",
  "data_snapshot": "snapshot-id",
  "runtime_snapshot": null
}
```

Keep these concepts distinct:

```text
image digest
    immutable build artifact

block snapshot
    durable application data

VM memory/device snapshot
    fast-continuation optimization

configuration digest
    runtime identity and policy
```

---

# 12. Weekly work-log template

Create `docs/weekly/week-NN.md` from this template:

```md
# Week NN — Title

## Status

- [ ] Reading complete
- [ ] Implementation complete
- [ ] Tests complete
- [ ] Notes complete
- [ ] Week closed

## Objective

One paragraph describing what this week is meant to make concrete.

## Reading notes

### Resource 1

- Key idea:
- Mechanism:
- Assumptions:
- Questions:

### Resource 2

- Key idea:
- Mechanism:
- Assumptions:
- Questions:

## Design before coding

### Invariants

1.
2.
3.

### Ownership and lifetime

- Who creates the object?
- Who owns it?
- When does ownership transfer?
- How is it destroyed or reset?

### Failure behaviour

- Invalid input:
- Out of memory:
- Partial completion:
- Reset:
- Snapshot:

## Implementation

### Commits

- `commit-hash` — summary

### Main decisions

1.
2.
3.

## Tests

| Test | Expected | Observed | Pass? |
|---|---|---|:---:|
| | | | [ ] |

## Measurements

| Metric | Value | Environment |
|---|---:|---|
| | | |

## What failed

- Failure:
- Root cause:
- Fix:
- Test added:

## What changed in my mental model

1.
2.
3.

## Open questions

- [ ]

## Exit artifacts

- `path/to/file`
```

---

# 13. Paper-review template

Create one file per paper in `docs/paper-reviews/`:

```md
# Paper title

## Citation

- Authors:
- Venue:
- Year:
- Link:

## Problem

What problem does the paper claim to solve?

## Context

What systems and assumptions existed when it was written?

## Core architecture

Describe the major components and their boundaries.

## Protection boundary

What prevents one workload or component from harming another?

## Host/platform ABI

What operations cross the boundary?

## Resource policy

Where are scheduling, memory, and I/O policies implemented?

## Application model

How are applications written, ported, linked, configured, and started?

## Compatibility

Which existing APIs or binaries are supported?

## Specialization

When and how is unused functionality removed?

## Measurements

What was measured? Are the baselines fair? What is not measured?

## Limitations

What does the paper not solve?

## Security implications

What new attack surfaces or removed surfaces result?

## Relevance to oc-uk

### Adopt

- 

### Reject

- 

### Investigate

- 

## Questions after reading

1.
2.
3.
```

---

# 14. Architecture-decision template

Create ADRs in `docs/architecture-decisions/`:

```md
# ADR-NNNN: Decision title

- Status: Proposed | Accepted | Superseded | Rejected
- Date: YYYY-MM-DD
- Owners:

## Context

What problem requires a decision?

## Decision

State the chosen design precisely.

## Alternatives considered

### Alternative A

Advantages:

- 

Disadvantages:

- 

### Alternative B

Advantages:

- 

Disadvantages:

- 

## Consequences

### Positive

- 

### Negative

- 

### Operational

- 

### Security

- 

## Invariants

1.
2.
3.

## Validation plan

- Tests:
- Benchmarks:
- Failure injection:

## Revisit when

List concrete conditions that would justify changing the decision.
```

---

# 15. Benchmark matrix

Maintain this table throughout the project rather than waiting until Week 60.

| Metric | Minimal Linux/QEMU | Unikraft | `oc-uk`/QEMU | `oc-uk`/Firecracker | `oc-uk`/`oc-vmm` |
|---|---:|---:|---:|---:|---:|
| ELF/image size | | | | | |
| Time to first instruction | | | | | |
| Time to allocator ready | | | | | |
| Time to network ready | | | | | |
| Time to application ready | | | | | |
| Host RSS at idle | | | | | |
| Guest allocated pages | | | | | |
| Peak heap | | | | | |
| TCP throughput | | | | | |
| HTTP requests/second | | | | | |
| p50 latency | | | | | |
| p95 latency | | | | | |
| p99 latency | | | | | |
| VM exits/request | | | | | |
| Interrupts/packet | | | | | |
| Snapshot size | | | | | |
| Snapshot creation time | | | | | |
| Restore-to-ready time | | | | | |

Record for every benchmark:

```text
Git commit
compiler version
build profile
host CPU
host kernel
QEMU/Firecracker/VMM version
vCPU count
memory size
network setup
block setup
request generator
sample count
warm-up policy
```

---

## First action

Start with **Week 01**, then create the Week 02 Nix/QEMU/GDB environment before writing kernel code.

The first major technical milestone is:

> A reproducibly built Rust `no_std` x86-64 ELF that boots through Limine, writes to COM1, reports its memory map, catches a deliberate page fault, and exits QEMU with a machine-readable result.

Do not move beyond that milestone until it has:

- [ ] Nix development environment;
- [ ] custom target configuration;
- [ ] custom linker script;
- [ ] linker map;
- [ ] separate RX/R/RW ELF segments;
- [ ] serial logging;
- [ ] GDB launch script;
- [ ] QEMU test exit;
- [ ] CI boot test;
- [ ] documented boot ABI.
