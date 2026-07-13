# Appendix H — Assessments, Design Reviews, and Graduation Rubric

## Phase reviews

At each phase gate, conduct a written design review. The reviewer may be another engineer or your future self using a recorded checklist.

### Review packet

```text
architecture diagram
public interfaces
invariants
unsafe code inventory
test matrix
known failures/non-goals
benchmark delta
one live demonstration
one deliberate failure demonstration
ADRs written in the phase
```

## Oral examination questions

### Machine and binary

- Explain every step from source code to `_start`.
- Draw sections and segments for your image.
- Explain one relocation.
- Demonstrate an ABI violation and its symptom.

### CPU and memory

- Draw a page-table walk for a concrete address.
- Explain the page-fault error code from a real failure.
- Show how a double-fault stack is selected.
- Demonstrate that code is not writable and heap is not executable.

### Execution

- Prove a task cannot be both runnable and blocked.
- Explain one acquire/release relationship.
- Reproduce and fix a lost wake-up.
- Explain what adding a second vCPU changes.

### Devices

- Trace descriptor ownership through one virtio request.
- Explain reset with an in-flight request.
- Show a malformed completion being rejected.
- Measure notifications and interrupts per operation.

### Networking/storage

- Trace one HTTP request from TAP to application and back.
- Explain a checksum/length failure test.
- Explain what `flush` guarantees in your stack.
- Restore from every prefix of a storage transaction or explain the tested crash model.

### Virtualization

- Explain a VM exit from guest instruction to VMM response.
- Build a VM through raw KVM calls.
- Show GVA→GPA→HVA reasoning for one buffer.
- Explain the CPU contract used for snapshots.

### Unikernel architecture

- Explain why the system is a library OS rather than small Linux.
- Identify the application ABI and platform ABI.
- Explain which Unix semantics are intentionally absent.
- Run one application under hosted, QEMU, and Firecracker backends.

### Security/operations

- Walk through the threat model for a malicious guest.
- Demonstrate VMM path/resource restrictions.
- Clone a snapshot twice and prove identity/entropy divergence.
- Explain data checkpoint versus runtime suspend.

## Rubric

Score each category from 0–4:

| Score | Meaning |
|---:|---|
| 0 | absent |
| 1 | demo works but mechanism/invariants unclear |
| 2 | mechanism understood, basic tests present |
| 3 | failure handling, documentation, and measurements present |
| 4 | robust, fuzzed/adversarially tested, portable across intended targets |

Categories:

```text
build reproducibility
ELF/ABI correctness
CPU/trap correctness
memory management
execution/concurrency
virtio/device correctness
networking
storage/checkpoints
KVM/VMM
library-OS architecture
application SDK/compatibility
security
observability/performance
Opencomputer integration
```

A credible graduation target is no category below 2, core safety/correctness categories at 3, and at least one specialty area at 4.

## Required demonstrations

1. **Cold boot** — signed image to application-ready under Firecracker.
2. **Fault report** — deliberate page fault with symbolized/bounded evidence.
3. **Network path** — packet capture plus guest/VMM trace for one HTTP request.
4. **Persistent state** — write, checkpoint, destroy, restore, verify.
5. **KVM VMM** — boot the same guest under `oc-vmm`.
6. **Malformed input** — fuzz/regression case rejected safely.
7. **Quiesce** — active service enters quiesced state within deadline.
8. **Cross-worker restore** — recreate network and exact storage on a compatible worker.
9. **Security boundary** — malicious descriptor cannot escape guest memory or exhaust host unboundedly.
10. **Benchmark** — reproducible comparison with declared baselines.

## Final written deliverables

```text
ARCHITECTURE.md
BOOT-ABI.md
APPLICATION-ABI.md
PLATFORM-ABI.md
ARTIFACT-FORMAT.md
CONTROL-PROTOCOL.md
SNAPSHOT-FORMAT.md
THREAT-MODEL.md
PORTING.md
PERFORMANCE.md
OPERATIONS.md
COMPATIBILITY.md
```

## Definition of mastery

Mastery does not require reproducing the ecosystem breadth of Linux, Firecracker, MirageOS, or Unikraft. It means you can:

- explain the mechanisms without hand-waving;
- implement a coherent subset;
- identify where production systems add complexity and why;
- test failure and adversarial paths;
- make evidence-based architecture choices;
- integrate the system into a real build/runtime lifecycle;
- know which guarantees you do and do not provide.
