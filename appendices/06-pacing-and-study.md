# Appendix F — Pacing, Full-Time Schedule, and Study Method

## Expected duration

The core curriculum contains 60 nominal weeks at 12–15 hours per week. That represents roughly 750–900 focused hours.

A full-time engineer with strong general programming and distributed-systems experience can compress it, but debugging and hardware details do not compress linearly.

Reasonable full-time ranges:

| Outcome | Focused time |
|---|---:|
| Read kernel code comfortably and complete xv6 labs | 6–10 weeks |
| Build a tested x86-64 learning kernel | 12–18 weeks cumulative |
| Build a small KVM VMM | 18–24 weeks cumulative |
| First networked application unikernel | 26–34 weeks cumulative |
| Firecracker/Opencomputer capstone | 34–44 weeks cumulative |
| Advanced stretch track | 12–24 additional weeks |

For an experienced CTO-level engineer working 40–50 hours per week, a realistic target for the 60-week core is **8–10 months**, with **10–14 months** for the advanced VMM, SDK, and platform goals. Treat these as planning ranges, not deadlines.

## Why full-time is not simply 3× faster

Low-level work includes long debugging episodes whose duration is not proportional to reading speed:

```text
wrong page-table bit
incorrect stack frame
missing memory barrier
stale descriptor after reset
interrupt acknowledgement race
VMM/guest address-domain confusion
snapshot state mismatch
```

When blocked, switching to a hosted reproducer, source trace, or specification review is more productive than increasing hours.

## Recommended full-time cadence

### Monday — model and plan

- Read the conceptual and OSDev material.
- Identify primary specification sections.
- Draw the mechanism and write invariants.
- Define the smallest demonstrable result.

### Tuesday–Wednesday — implementation

- Build in small commits.
- Add hosted tests before VM integration where possible.
- Keep one known-good boot path available.

### Thursday — integration and adversarial tests

- Boot in QEMU/Firecracker or run in `oc-vmm`.
- Add malformed input and failure injection.
- Capture traces and metrics.

### Friday — review and documentation

- Clean APIs and unsafe contracts.
- Run all tests from a clean environment.
- Write weekly log and ADRs.
- Record benchmark and remaining risks.
- Decide whether exit criteria are actually met.

## Fast track without shallow learning

You may shorten the course by:

- reading selected textbook sections rather than entire chapters;
- completing the hardest/most relevant xv6 labs rather than every optional exercise;
- using a bootloader rather than writing firmware;
- integrating smoltcp instead of writing production TCP;
- remaining single-vCPU through the core;
- using a read-only archive/custom log instead of a full filesystem;
- targeting source compatibility rather than Linux binary compatibility;
- using Firecracker for production rather than finishing a hardened VMM first.

Do not skip:

```text
ELF/linker/ABI
page tables and faults
interrupt/trap frames
memory ordering
virtqueue ownership
raw KVM VM creation
snapshot consistency
security boundaries
```

These are the concepts most likely to create hidden architectural mistakes.

## Part-time modes

### 10–15 hours/week

Use the original 60-week schedule. Keep one evening for reading, one for implementation, and a larger weekend block for integration.

### 5–8 hours/week

Treat each nominal week as two calendar weeks. Expect 24–30 months. Avoid long gaps; low-level context is expensive to reload.

### Intensive sabbatical

At 50–60 hours/week, use six-day cycles but reserve one day for consolidation. More than two consecutive weeks without documentation and review tends to create brittle code and forgotten reasoning.

## When to repeat a week

Repeat or extend a week when:

- you cannot explain the end-to-end path without source;
- tests cover only success;
- a subsystem works only with logging/timing changes;
- the API exposes target-specific details upward;
- unsafe assumptions are undocumented;
- you cannot reproduce a failure from stored evidence;
- exit criteria rely on “seems to work.”

Do not repeat merely to polish style before the next mechanism can test the architecture.

## Monthly review

Every four weeks answer:

1. What can the system do now that it could not do before?
2. Which mechanism do I still explain vaguely?
3. Which tests are only integration tests and should move to hosted tests?
4. Which target-specific assumptions leaked into common code?
5. Which benchmark changed and why?
6. Which non-goal am I accidentally implementing?
7. Is the Opencomputer boundary becoming clearer or more coupled?

## Suggested milestone demonstrations

### Month 1–2

- ELF inspector and freestanding entry.
- xv6 syscall/page-table/trap modifications.

### Month 3–4

- Bare-metal faults, memory allocator, executor.
- ICMP/UDP over virtio-net.

### Month 5–6

- HTTP unikernel and virtio-block.
- Raw KVM VMM booting the guest.

### Month 7–8

- Native SDK and one C port.
- Firecracker boot/control/snapshot.

### Month 9–10

- Opencomputer artifact/worker/checkpoint capstone.
- Security, fuzzing, and comparative benchmark report.
