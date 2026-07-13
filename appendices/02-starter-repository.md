# Appendix B — Starter Repository and Build Scaffolds

This appendix provides scaffolds, not a complete kernel. Keep each file small enough to understand and replace.

## Suggested tree

```text
unikernel-lab/
├── flake.nix
├── Cargo.toml
├── rust-toolchain.toml
├── justfile
├── linker/
│   └── x86_64.ld
├── scripts/
│   ├── run-qemu
│   ├── debug-qemu
│   ├── inspect-image
│   └── vm-test
├── labs/
├── xv6-labs/
├── oc-kernel/
├── oc-vmm/
├── oc-uk/
├── apps/
├── docs/
│   ├── architecture-decisions/
│   ├── notes/
│   └── paper-reviews/
├── tests/
├── fuzz/
└── benches/
```

## Rust toolchain pin

```toml
# rust-toolchain.toml
[toolchain]
channel = "nightly-YYYY-MM-DD"
components = ["rust-src", "llvm-tools-preview", "clippy", "rustfmt"]
profile = "minimal"
```

Use the oldest/nightly features actually required. Reevaluate whether unstable features remain necessary as Rust evolves.

## Workspace scaffold

```toml
# Cargo.toml
[workspace]
resolver = "2"
members = [
  "labs/address-types",
  "labs/elf-inspector",
  "oc-kernel",
  "oc-vmm",
  "oc-uk/core",
  "oc-uk/platform-hosted",
  "oc-uk/platform-x86_64",
  "apps/hello",
]

[profile.release]
panic = "abort"
lto = "thin"
codegen-units = 1
```

Do not blindly optimize for size from day one. Keep debug symbols in a separate artifact even when the runtime image is stripped.

## Nix development shell scaffold

```nix
{
  description = "Unikernel engineering laboratory";

  inputs.nixpkgs.url = "github:NixOS/nixpkgs/<pinned-revision>";

  outputs = { self, nixpkgs }:
    let
      system = "x86_64-linux";
      pkgs = import nixpkgs { inherit system; };
    in {
      devShells.${system}.default = pkgs.mkShell {
        packages = with pkgs; [
          rustup
          clang
          lld
          llvm
          binutils
          nasm
          qemu
          gdb
          just
          python3
          socat
          tcpdump
          jq
          pkg-config
        ];

        shellHook = ''
          export RUST_BACKTRACE=1
          export CARGO_TERM_COLOR=always
        '';
      };
    };
}
```

Pin a real revision. Package names may differ across nixpkgs revisions; treat this as a starting shape.

## `justfile` scaffold

```make
set shell := ["bash", "-euo", "pipefail", "-c"]

build:
    cargo build -p oc-kernel

run: build
    ./scripts/run-qemu

debug: build
    ./scripts/debug-qemu

test:
    cargo test --workspace --exclude oc-kernel
    ./scripts/vm-test

inspect: build
    ./scripts/inspect-image target/.../oc-kernel

fmt:
    cargo fmt --all -- --check

lint:
    cargo clippy --workspace --all-targets -- -D warnings
```

## Linker script scaffold

Use the more detailed script in Chapter 1 as a basis. Export at least:

```text
__image_start/__image_end
__text_start/__text_end
__rodata_start/__rodata_end
__data_start/__data_end
__bss_start/__bss_end
```

Validate program headers after every build.

## Address newtypes

```rust
#[derive(Clone, Copy, Eq, PartialEq, Ord, PartialOrd, Hash)]
#[repr(transparent)]
pub struct GuestPhysAddr(u64);

impl GuestPhysAddr {
    pub const fn new(value: u64) -> Self {
        Self(value)
    }

    pub fn checked_add(self, bytes: u64) -> Option<Self> {
        self.0.checked_add(bytes).map(Self)
    }

    pub const fn raw(self) -> u64 {
        self.0
    }
}
```

Add validated constructors for canonical virtual addresses and conversions that require an explicit mapping context.

## Emergency serial interface

```rust
pub trait EmergencyWriter {
    fn write_byte(&self, byte: u8);

    fn write_bytes(&self, bytes: &[u8]) {
        for &byte in bytes {
            self.write_byte(byte);
        }
    }
}
```

The low-level implementation may require unsafe port I/O, but the formatting and logging layers should remain safe.

## VM test envelope

A host runner should produce JSON even on failure:

```json
{
  "test": "page-fault-report",
  "result": "pass",
  "guest_exit": 0,
  "duration_ms": 82,
  "image_digest": "sha256:...",
  "serial_log": "artifacts/page-fault-report.log",
  "vmm_stderr": "artifacts/page-fault-report.vmm.log"
}
```

Use a hard timeout and kill the entire process group.

## GDB command file

```gdb
set pagination off
set confirm off
set disassembly-flavor intel
target remote :1234
symbol-file result/oc-kernel.elf
break _start
```

Add helper commands later for page-table walks and trap-frame printing.

## ELF inspection script expectations

`inspect-image` should fail if:

```text
wrong architecture/class
entry not executable
load ranges overlap
filesz > memsz
range arithmetic overflows
RWX segment exists
required ELF note absent
image exceeds supported guest memory
```

## Platform traits

```rust
pub trait Platform {
    fn memory_map(&self) -> &[MemoryRegion];
    fn monotonic_now(&self) -> Instant;
    fn fill_entropy(&self, out: &mut [u8]) -> Result<(), EntropyError>;
    fn devices(&self) -> &[DeviceDescriptor];
    fn shutdown(&self, reason: ShutdownReason) -> !;
}
```

Keep the trait narrow. Do not add a method merely to expose a convenient target-specific detail.

## Suggested first commits

```text
chore: initialize Nix and Cargo workspace
build: add deterministic QEMU commands
test: add host timeout and VM result envelope
feat: add typed address helpers
feat: add strict ELF inspector
build: add custom linker script and permission check
feat: add serial-only freestanding entry
```
