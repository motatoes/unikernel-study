# Appendix E — Glossary

**ABI — Application Binary Interface**  
The binary-level contract for calling conventions, data layout, symbols, object formats, and process/application entry.

**API — Application Programming Interface**  
A source-level interface. An API can remain stable while its ABI changes, or vice versa.

**Application checkpoint**  
A durable, application-defined representation of state, normally portable across fresh runtime instances when versions are compatible.

**APIC — Advanced Programmable Interrupt Controller**  
x86 interrupt-controller architecture used for local and system interrupt routing.

**COW — Copy on write**  
Sharing an object until a writer creates a private copy. Used in page sharing and block snapshot overlays.

**CR0/CR2/CR3/CR4**  
x86 control registers. CR2 records the page-fault linear address; CR3 identifies the active page-table root.

**DMA — Direct Memory Access**  
Device access to memory without CPU copying each byte. In a VM, a virtio descriptor normally contains guest-physical addresses.

**ELF — Executable and Linkable Format**  
Object/executable format used for sections, symbols, relocations, and loadable segments.

**EPT/NPT**  
Intel Extended Page Tables and AMD Nested Page Tables, the second translation from guest physical to host physical memory.

**Eventfd**  
Linux event-notification file descriptor commonly used by VMMs and device backends.

**GDT — Global Descriptor Table**  
x86 descriptor table containing code/data descriptors and the TSS descriptor even in long-mode kernels.

**GPA/GVA/HPA/HVA**  
Guest physical, guest virtual, host physical, and host virtual address domains.

**IDT — Interrupt Descriptor Table**  
x86 table mapping interrupt/exception vectors to entry gates.

**IST — Interrupt Stack Table**  
TSS mechanism selecting a dedicated stack for specified interrupt/exception entries.

**KVM**  
Linux kernel virtualization API/execution facility. A userspace VMM supplies memory, images, devices, and lifecycle management.

**Library OS**  
OS functionality linked as libraries with an application, often over a narrow host/platform interface.

**MMIO — Memory-Mapped I/O**  
Device registers accessed through addresses in a memory space with volatile and ordering requirements.

**Monotonic clock**  
A time source that does not move backward within its defined lifecycle, suitable for deadlines.

**MSI/MSI-X**  
Message-signaled interrupt mechanisms used by PCI/PCIe devices.

**Nix closure**  
A build result plus all transitively required store paths, identified through content-derived paths and dependency references.

**NX — No execute**  
Page permission preventing instruction fetch from data memory.

**OCI**  
Open Container Initiative specifications for content-addressed images, runtimes, and distribution. OCI artifact infrastructure can distribute non-container artifacts using custom media types.

**Page fault**  
x86 exception caused by a translation/protection problem; CR2 and the error code describe the access.

**Paravirtualization**  
A guest interface deliberately designed for virtualization rather than emulating arbitrary physical hardware. Virtio devices are paravirtualized.

**POSIX shim**  
A compatibility layer implementing a selected subset of POSIX/Unix interfaces over the native runtime.

**Quiesce**  
Transition to a state where no new work is admitted and in-flight state is drained or stabilized for checkpoint/shutdown.

**Ring buffer**  
Bounded circular queue. Virtqueues use ring structures to exchange descriptor heads and completions.

**SMP — Symmetric Multiprocessing**  
Execution with multiple CPUs/vCPUs sharing memory and OS responsibilities.

**Snapshot**  
Captured runtime or storage state. In this handbook, “VM runtime snapshot” is distinguished from durable application checkpoint.

**Solo5 tender**  
A userspace monitor/binding component providing a narrow execution interface for supported library OSes.

**TAP**  
Host virtual network interface carrying Ethernet frames to/from a userspace VMM.

**TLB — Translation Lookaside Buffer**  
CPU cache of address translations and permissions.

**TSS — Task State Segment**  
x86 structure used in long mode for stack pointers, IST entries, and I/O permissions; not normally for hardware task scheduling.

**Unikernel**  
A single-purpose application image statically composed with the OS facilities it needs, commonly running in one address space.

**VMM — Virtual Machine Monitor**  
Userspace and/or privileged software that configures and runs VMs, handles exits, and supplies devices/lifecycle.

**VM exit**  
Transfer from guest execution to the hypervisor/KVM/VMM because of a configured event or operation.

**Virtio**  
Standard family of paravirtualized device interfaces with feature negotiation, transports, and virtqueues.

**Virtqueue**  
Shared descriptor/ring mechanism through which a virtio driver and device exchange buffers and completions.

**Vsock**  
Socket address family for host/guest or VM/VM communication without ordinary IP networking.

**W^X**  
Policy that memory is not simultaneously writable and executable.
