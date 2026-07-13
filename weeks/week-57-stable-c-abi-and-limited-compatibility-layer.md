> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 16 — Application SDKs, C ABI, and Limited POSIX Compatibility](../chapters/16-application-abi-posix.md)

---

# Week 57 — Stable C ABI and limited compatibility layer

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
