# Chapter 16 — Application SDKs, C ABI, and Limited POSIX Compatibility

## Purpose

A library OS becomes useful when applications can target it through stable, comprehensible interfaces. The first interface should match the runtime's actual model. A compatibility layer should then be added deliberately for selected applications, with every semantic gap documented.

## Learning objectives

You should be able to:

- design a versioned native application API;
- define a stable C ABI independent of Rust layout;
- distinguish source compatibility, ABI compatibility, and binary compatibility;
- implement a minimal libc/syscall shim for one application;
- inventory hidden Unix assumptions in existing software;
- define unsupported behavior honestly;
- build a repeatable application porting workflow.

## Native application model

A native Rust application should receive capabilities rather than ambient global access:

```rust
pub trait UnikernelApp {
    fn init(config: &AppConfig) -> Result<Self, AppError>
    where
        Self: Sized;

    fn run(&mut self, runtime: &Runtime) -> Result<(), AppError>;

    fn quiesce(&mut self, reason: QuiesceReason)
        -> Result<(), AppError>;

    fn shutdown(&mut self, reason: ShutdownReason);
}
```

An async facade may be more ergonomic:

```rust
#[oc_uk::main]
async fn main(ctx: AppContext) -> Result<()> {
    let listener = ctx.net.bind_tcp("0.0.0.0:8080").await?;
    loop {
        let connection = listener.accept().await?;
        ctx.tasks.spawn(handle(connection));
    }
}
```

The macro should generate a stable entry shim, not hide lifecycle semantics.

## API versioning

Version separately:

```text
image manifest format
boot/platform ABI
native application API
C application ABI
control protocol
snapshot format
```

Changing one should not force all others to change. Define compatibility rules and reject unsupported major versions at build or launch.

## Stable C ABI

Rust-native trait layouts are not durable ABIs. Expose C-compatible fixed-width structures and function tables:

```c
#define OC_UK_APP_ABI_V1 1

struct oc_uk_slice {
    const uint8_t *ptr;
    size_t len;
};

struct oc_uk_runtime_v1 {
    uint32_t abi_version;
    int (*log)(uint32_t level, struct oc_uk_slice msg);
    int (*clock_now_ns)(uint64_t *out);
    int (*getrandom)(void *buf, size_t len);
    /* additional versioned functions */
};

int oc_uk_main_v1(const struct oc_uk_runtime_v1 *runtime,
                  struct oc_uk_slice config);
```

Rules:

- use explicit representation and integer widths;
- carry lengths with pointers;
- avoid exposing internal allocator ownership accidentally;
- define error codes and thread-safety;
- version function tables;
- validate application-provided pointers when crossing unsafe FFI code, even in one address space.

## Compatibility levels

### Source compatibility

Application source can be recompiled against your headers/libraries with changes limited to configuration or build scripts.

### ABI compatibility

An already compiled object or library can link against your implementation because calling conventions and symbol contracts match.

### Binary compatibility

An existing executable can run without relinking, often requiring ELF loading, dynamic-linker behavior, syscall ABI, thread-local storage, auxiliary vectors, and substantial Linux semantics.

For the first project, target source compatibility for one C application. Binary compatibility is a separate major project.

## Syscall/libc ledger

Maintain a table:

| Interface | Status | Semantic notes | Tests | Required by |
|---|---|---|---|---|
| `write` | implemented | stdout/stderr to structured log | ... | app A |
| `clock_gettime` | partial | monotonic only initially | ... | libc |
| `mmap` | partial | anonymous private mappings only | ... | allocator |
| `fork` | unsupported | no process model | n/a | none |

Never label the system “POSIX compatible” without specifying the tested profile and differences.

## First compatibility surface

A practical starting set may include:

```text
exit / abort
read / write / close
clock_gettime / nanosleep
getrandom
mmap / munmap
poll-like wait primitive
socket / bind / listen / accept
send / recv / shutdown
basic environment/config access
```

Implement only what the selected application and libc require. Stubbed functions that return false success are dangerous; return explicit unsupported errors.

## Hidden application assumptions

Existing software may assume:

- filesystem paths and current directory;
- dynamic libraries and plugins;
- environment variables;
- locale/timezone databases;
- `/proc`, `/dev`, and `/etc`;
- users, groups, permissions;
- signals;
- `fork`/`exec`;
- thread-local storage;
- wall-clock time;
- DNS resolver behavior;
- file locking;
- entropy availability;
- durable rename/fsync semantics.

Instrument unresolved symbols and unsupported calls to create an empirical porting inventory.

## Porting workflow

1. Select a small, bounded application.
2. Build it statically on Linux and inspect dependencies.
3. Disable optional features and plugins.
4. Compile against your C headers and shim.
5. Resolve link errors one capability at a time.
6. Run hosted tests against the same shim.
7. Boot in QEMU and capture unsupported calls.
8. Add semantics plus tests, not just symbol stubs.
9. Compare behavior with Linux using the application's own test suite.
10. Record the compatibility profile.

SQLite with a custom VFS is a useful advanced target because it forces explicit storage and locking semantics. A small C HTTP server is simpler for the first port.

## Memory allocation across the ABI

Define who allocates and who frees. Avoid returning runtime-owned pointers with ambiguous lifetime. Options include:

- caller-provided buffers;
- paired allocate/free functions within one ABI;
- opaque handles;
- immutable borrowed slices valid only for the call.

Never free memory through a different allocator than the one that created it.

## Threading and TLS

If the application expects pthreads, decide whether to:

- map them to cooperative tasks;
- implement real preemptive threads;
- restrict to one thread;
- reject the application.

Thread-local storage requires compiler/runtime support and context-switch integration. Do not claim thread support based solely on a `pthread_create` symbol.

## Debugging playbook

### Application links but fails immediately

Check constructors, TLS, stack alignment, environment/auxiliary data, panic/abort behavior, and assumptions made before `main`.

### Works with one connection, fails under concurrency

Inspect shim thread-safety, blocking semantics, allocator synchronization, socket wakeups, and application assumptions about preemption.

### Data corruption across FFI

Check structure layout, alignment, integer widths, ownership, and C compiler flags. Add ABI layout tests compiled from both C and Rust.

### Stubbed function causes strange behavior later

Replace permissive stubs with explicit unsupported errors. Applications often interpret a successful return as a strong semantic guarantee.

## Exercises

1. Define `oc_uk_main_v1` and call a C hello-world application.
2. Compile layout assertions in both C and Rust.
3. Port a small C HTTP service with a documented syscall ledger.
4. Add `clock_gettime`, `getrandom`, and anonymous `mmap` with conformance tests.
5. Build SQLite with a custom in-memory VFS, then extend it to your block log.
6. Compare native SDK and POSIX-shim image size, boot time, and attack surface.

## Review questions

1. Why is a Rust trait not a stable C ABI?
2. How do source, ABI, and binary compatibility differ?
3. Why is a successful no-op stub dangerous?
4. Which ownership information must cross an FFI boundary?
5. What does pthread compatibility require beyond function names?
6. How should unsupported semantics be communicated to users?

## Opencomputer connection

Opencomputer should advertise build targets explicitly: native `oc-uk` SDK, source-ported C ABI, or Linux/OCI runtime. A workload should not silently fall back between them. Build reports should show selected compatibility libraries, unsupported interfaces, image size contribution, and the exact ABI profile so users can choose the right execution model.
