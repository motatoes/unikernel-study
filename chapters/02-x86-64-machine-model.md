# Chapter 2 — The x86-64 Machine Model

## Purpose

A kernel or unikernel cannot rely on the host operating system to maintain CPU state. You must know which state exists, who initializes it, what the architecture guarantees, and what causes control to transfer into your handlers.

## Learning objectives

You should be able to:

- identify general, control, segment, model-specific, and extended-state registers;
- explain long mode, canonical addresses, and privilege rings;
- describe the roles of the GDT, TSS, IDT, and local APIC;
- distinguish faults, traps, aborts, and external interrupts;
- reason about the state saved by hardware and the state saved by software;
- create a reliable register dump for postmortem debugging;
- explain how the machine model changes under virtualization.

## Architectural state

Useful categories include:

### General execution state

```text
RAX RBX RCX RDX
RSI RDI RBP RSP
R8–R15
RIP RFLAGS
```

The ABI assigns conventions to these registers, but exceptions and context switches must preserve whatever state your runtime promises.

### Control state

```text
CR0  operating mode and protection controls
CR2  faulting linear address for page faults
CR3  root of the active page-table hierarchy
CR4  architectural feature controls
EFER long-mode and syscall-related controls
```

### Descriptor state

```text
GDTR → Global Descriptor Table
IDTR → Interrupt Descriptor Table
TR   → Task State Segment descriptor
```

Long mode reduces the importance of segmentation for ordinary addressing, but the GDT and TSS still matter for privilege transitions, stack selection, and interrupt-stack-table entries.

### Extended state

Floating-point, SIMD, and other architectural extensions have separate state. A scheduler that allows tasks to use them must define how that state is initialized, saved, and restored. Ignoring it can create cross-task corruption or information leakage.

## Privilege and control transfer

A unikernel often executes all guest code at ring 0 because the application and library OS are statically combined. That removes user/kernel transitions inside the guest, but it does not remove:

- page permissions;
- exception delivery;
- interrupt delivery;
- VM exits;
- the hypervisor security boundary;
- the need to treat untrusted input as hostile.

Single-address-space does not mean “no protection.” It means protection must come from language safety, memory permissions, narrow capabilities, and the hypervisor rather than a process boundary inside the guest.

## Canonical addresses

Not every 64-bit value is a valid virtual address. x86-64 requires unused high bits to be a sign extension of the highest implemented virtual-address bit. Address wrappers should validate canonicality and use checked arithmetic. A wrapped or noncanonical pointer can cause a general-protection fault rather than a page fault.

## Descriptor tables

### GDT

For a first long-mode kernel, the GDT normally contains a minimal set of code/data descriptors plus a TSS descriptor. Keep it small and document selector values.

### TSS

In long mode, the TSS is chiefly useful for:

- privilege-level stack pointers;
- interrupt stack table entries;
- I/O permission bitmap configuration.

Use a dedicated interrupt stack for double faults and possibly other catastrophic exceptions. A stack-overflow-induced page fault cannot safely use the already-corrupted normal stack.

### IDT

Each IDT entry describes a handler, selector, gate type, privilege level, presence, and optional IST index. The assembly entry path must normalize differences between exceptions that push an error code and those that do not.

## Exception taxonomy

A practical engineering view:

- **Fault**: reported before completion; the instruction may be retried after correction, as with a recoverable page fault.
- **Trap**: reported after an instruction, often used for debugging.
- **Abort**: severe condition with no reliable restart point.
- **External interrupt**: asynchronous event delivered by interrupt hardware.

Your handler API should preserve the vector, optional error code, instruction pointer, stack pointer, flags, and fault-specific state such as CR2.

## Trap-frame design

A good trap frame has one canonical layout regardless of vector. The assembly stub can push a synthetic zero error code for exceptions that do not receive one, then save registers in a fixed order.

```text
high address
┌────────────────────────────┐
│ hardware-saved stack state │
├────────────────────────────┤
│ vector                     │
│ normalized error code      │
├────────────────────────────┤
│ general registers          │
├────────────────────────────┤
│ optional extended metadata │
└────────────────────────────┘
low address
```

Do not let Rust's default struct layout define an assembly ABI. Use explicit representation and static offset checks.

## Interrupt handling rules

Keep first-level handlers short:

```text
validate vector
acknowledge or mask source when required
capture minimal event state
enqueue deferred work
return
```

Avoid allocation, blocking, complex formatting, and nested lock acquisition in interrupt context until you have designed explicit support for them.

## Virtualization perspective

Under KVM, most guest instructions execute directly on the CPU in non-root guest mode. Selected operations, device accesses, or configured events cause VM exits. The guest still observes architectural exceptions and interrupts, but the VMM may synthesize device state and inject interrupts. Debugging therefore requires tracing both guest state and VMM exit handling.

## Debugging playbook

### Triple fault / VM reset

Typical causes:

- invalid IDT entry;
- exception handler faults before a valid double-fault path exists;
- bad stack or TSS;
- returning with a mismatched frame;
- loading an invalid GDT/IDT limit or base.

Run QEMU without automatic reboot, enable guest-error logging, attach GDB before loading the table, and single-step the faulting instruction.

### Wrong exception vector

Check stub generation, error-code normalization, gate indexing, and whether the CPU delivered a secondary exception while entering the first handler.

### Handler returns into nonsense

Check stack layout, register push/pop order, stack alignment, and whether the return instruction matches the privilege transition and frame shape.

## Exercises

1. Generate all IDT stubs from one macro and verify their binary layout.
2. Trigger divide error, invalid opcode, breakpoint, general-protection fault, page fault, and double fault.
3. Add a guard page below a deliberately small stack and produce a reliable double-fault report.
4. Save and restore an artificial CPU context in userspace, then port the layout to the kernel.
5. Add static assertions for every trap-frame field offset consumed by assembly.

## Review questions

1. Why does a long-mode kernel still need a GDT?
2. Why is the TSS useful even without hardware task switching?
3. Which exceptions carry hardware error codes?
4. Why can a stack overflow turn an ordinary page fault into a double fault?
5. What additional state must be considered once tasks use SIMD?
6. Why does ring 0 application execution not eliminate security engineering?

## Opencomputer connection

Reliable fault reports are essential in a remote execution platform. `oc-uk` should emit a compact machine-readable crash record containing image digest, build ID, vCPU, vector, error code, RIP, stack bounds, selected registers, and a bounded backtrace. The worker should capture this record before tearing down the VM and attach it to the sandbox lifecycle event.
