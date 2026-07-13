# Chapter 6 — Boot, Build, Test, and Debug Bare-Metal Code

## Purpose

The earliest kernel code runs in an environment where ordinary diagnostics do not exist. A reproducible build, serial output, machine-readable test exit, linker inspection, and debugger attachment are not optional tooling; they are the foundation that makes every later subsystem possible.

## Learning objectives

You should be able to:

- define a reproducible Nix-based systems laboratory;
- boot a freestanding x86-64 ELF through a documented protocol;
- initialize serial output before complex runtime code;
- run automated VM tests with deterministic pass/fail reporting;
- attach GDB before the first guest instruction;
- inspect QEMU logs and CPU state after fatal faults;
- minimize a failure into a reproducible test.

## Reproducible environment

Pin the compiler, target components, linker, QEMU, GDB, binutils, and helper tools. A useful development shell contains:

```text
rustc cargo rust-src llvm-tools
clang lld binutils nasm
qemu-system-x86_64 gdb
python socat tcpdump jq just
```

Nix should define the tool environment, while Cargo manages Rust packages. Keep the boundary clear: Nix pins external tools and system dependencies; Cargo defines Rust crates and features.

## Repository commands

A new contributor should need only:

```bash
nix develop
just test
just run
just debug
just inspect
```

Recommended commands:

```text
build       compile the selected image
run         boot with serial attached
debug       boot paused and start/print GDB instructions
test        run hosted and VM tests
inspect     print ELF headers, segments, symbols, and size
bench       run selected benchmark
fmt         format all supported languages
lint        clippy/static checks/shell checks
clean       remove generated artifacts
```

## Boot protocol choice

Use an existing bootloader or direct-kernel protocol for the exploratory kernel. The goal is to learn kernel mechanisms, not firmware parsing. Your boot interface should provide or allow discovery of:

- memory map;
- image bounds;
- command line or configuration;
- framebuffer only if desired;
- ACPI or platform tables if needed;
- modules/embedded files;
- an initial stack and CPU mode contract.

Immediately normalize protocol-specific data into your own `BootInfo` type. Do not let bootloader structs leak across the kernel.

## Serial first

COM1 output is simple, scriptable, and visible in headless CI. Initialize it before the heap, scheduler, or rich logger. Keep a minimal emergency write path that does not allocate or take ordinary locks.

A layered design:

```text
raw UART byte output
    ↓
nonallocating emergency formatter
    ↓
structured logger after initialization
    ↓
host log collector
```

Avoid depending on VGA or a graphical console for correctness.

## Machine-readable VM tests

A test VM should terminate with a status the host can interpret. Under QEMU, common approaches include a dedicated debug-exit device or a controlled shutdown protocol. Whichever mechanism you use, define:

```text
success code
failure code
panic code
infrastructure timeout
unexpected reset
```

The host runner must distinguish a guest test failure from QEMU failing to start.

## GDB workflow

Start QEMU paused with a GDB server, then load symbols from the exact ELF used to boot. Maintain a checked-in GDB command file:

```gdb
set pagination off
set disassembly-flavor intel
target remote :1234
symbol-file result/oc-kernel.elf
break _start
continue
```

Useful operations:

- inspect `rip`, `rsp`, `cr0`, `cr2`, `cr3`, `cr4`;
- walk stack memory;
- inspect page-table entries;
- disassemble around `rip`;
- set hardware watchpoints;
- single-step table loads and control-register writes.

Be careful with source-level stepping through optimized code. Assembly and register state are the final truth.

## QEMU diagnostic modes

Use targeted logging rather than enabling everything permanently. Useful categories include guest errors, interrupt activity, MMU behavior, and CPU reset state. Disable reboot after fatal faults so evidence remains available. Record the complete QEMU command line in test output.

## Layered test strategy

```text
hosted unit tests
    ↓
property/fuzz tests
    ↓
small VM subsystem tests
    ↓
full boot integration tests
    ↓
load/failure tests
```

Parsing, allocators, queue logic, network protocol validation, manifest handling, and snapshot decoding should be tested in normal processes whenever possible. Reserve VM tests for architecture and device behavior.

## Reproducibility record

Every benchmark and fault report should include:

```text
git commit
Rust toolchain
linker version
QEMU/VMM version
host kernel
host CPU model
build profile
image digest
VM configuration
```

Without this data, performance changes and low-level failures are difficult to reproduce.

## Debugging playbook

### No serial output

Confirm QEMU serial routing, I/O port permissions/implementation, UART initialization, entry point, stack, and whether the CPU reset before the first write. Use a one-byte write in assembly before Rust.

### GDB breakpoints appear at wrong addresses

Ensure the symbol file matches the booted artifact and account for higher-half or relocation offsets. Compare `readelf -l`, the loader's chosen address, and runtime RIP.

### CI hangs

Add host-side timeouts, capture serial to an artifact, disable interactive devices, report QEMU stderr, and map every exit status. A timeout should produce the last N kilobytes of guest logs and configuration.

### Test passes locally but not under Nix/CI

Check undeclared tools, host CPU assumptions, KVM availability, architecture-specific compiler flags, network privileges, and reliance on `/tmp` or current directory.

## Exercises

1. Create a VM test that deliberately passes, fails, panics, and hangs; verify the runner classifies all four.
2. Boot with one-byte assembly serial output before any Rust code.
3. Add `scripts/inspect-image` that rejects unsafe segment permissions.
4. Reproduce a page fault entirely from saved CI artifacts.
5. Run the same guest with TCG and KVM and record behavioral/performance differences.
6. Create a minimized boot test containing only startup, serial, and shutdown.

## Review questions

1. Why should boot-protocol structures be normalized immediately?
2. Which tests should run outside a VM?
3. Why is a serial logger still needed after adding vsock?
4. What makes a VM test result machine-readable?
5. Why can source-level GDB views be misleading under optimization?
6. Which environment details are essential in a benchmark record?

## Opencomputer connection

The worker runtime needs the same discipline at scale: deterministic command construction, exact image/configuration identity, bounded startup time, serial/control log capture, explicit readiness, and categorized termination. The local test runner is the prototype of the production lifecycle supervisor.
