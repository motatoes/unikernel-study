# Chapter 5 — OS Interfaces Through xv6

## Purpose

Before removing processes, users, system calls, and a general filesystem from your own system, you must understand what those abstractions accomplish. xv6 is compact enough to trace end to end and complete enough to show the interactions among traps, page tables, scheduling, file descriptors, filesystems, locks, and devices.

## Learning objectives

You should be able to:

- trace a user call through a trap into the kernel and back;
- explain process state, kernel stacks, context switching, and scheduling;
- describe page faults, copy-on-write, and address-space teardown;
- explain sleeping, wake-up, and lost-wake-up prevention;
- trace file operations through inodes, buffer cache, logging, and block I/O;
- identify which Unix abstractions a first unikernel can eliminate;
- separate teaching simplifications in xv6 from universal OS principles.

## Read xv6 as a set of complete paths

Do not read the source file by file. Trace operations across files.

### System-call path

```text
user wrapper
  → architecture trap instruction
  → trampoline entry
  → trap-frame save
  → syscall number dispatch
  → implementation
  → trap return
  → user continuation
```

Questions to answer:

- Which stack is active at each stage?
- Which page table is active?
- Where are arguments validated?
- What happens if a pointer crosses an unmapped page?
- Which state is trusted and which is copied?

### Scheduling path

```text
running process
  → blocks, yields, or is preempted
  → kernel context switch
  → scheduler context
  → choose runnable process
  → switch to process kernel context
  → return toward user mode
```

Separate the architecture-specific context switch from policy. This separation will later guide your `oc-uk` executor and `oc-vmm` vCPU loop.

### File-write path

```text
write(fd, buffer, n)
  → file object
  → inode operation
  → block mapping
  → buffer cache
  → log transaction
  → block driver
```

Study where locks are held, where code may sleep, and what makes a transaction crash-consistent.

## What xv6 teaches well

xv6 is especially useful for:

- explicit invariants;
- readable trap and syscall paths;
- small page-table code;
- process lifecycle;
- lock and sleep/wakeup interaction;
- inode and logging structure;
- complete source-level tracing.

## What xv6 does not represent fully

Do not infer production design directly from xv6. It simplifies:

- scalability;
- security hardening;
- device breadth;
- complex VM policies;
- network stack behavior;
- NUMA;
- asynchronous I/O;
- production filesystems;
- observability;
- compatibility.

The correct question is not “How do I copy xv6?” but “Which invariant or mechanism is xv6 making visible?”

## Process machinery versus unikernel machinery

A conventional process model provides:

- separate virtual address spaces;
- privilege transitions;
- user pointer validation;
- process identifiers and lifecycle;
- signals and exit status;
- file descriptor tables;
- resource accounting;
- isolation among applications.

A single-application unikernel may eliminate most of these internally. However, the Opencomputer platform still needs workload identity, resource accounting, lifecycle state, and isolation—those responsibilities move to the VMM and control plane rather than disappearing.

## Page faults and copy-on-write

The copy-on-write lab is valuable because it forces precise ownership reasoning:

```text
parent and child PTEs point to one frame
    ↓
write permission removed
    ↓
write fault
    ↓
validate COW mapping
    ↓
allocate/copy or reuse based on reference count
    ↓
update PTE and invalidate translation
```

The same style of reasoning applies to immutable image pages, snapshot sharing, and copy-on-write workspace disks in Opencomputer.

## Sleep and wake-up

The classic lost-wake-up pattern occurs when a task checks a condition, an event occurs, and then the task sleeps after the wake-up has already been sent. xv6's sleep/wakeup design demonstrates why condition checking and sleep registration must be coordinated under a lock.

Translate the lesson into your executor:

```text
observe condition
register interest
recheck condition
park task
```

The exact API differs, but the race does not.

## Filesystem and logging lessons

The xv6 log illustrates transaction boundaries and recovery rather than a production filesystem. Focus on:

- which writes form one invariant-preserving unit;
- why metadata ordering matters;
- how replay decides whether a transaction committed;
- why cache state and durable state differ;
- what application consistency requires beyond filesystem consistency.

For a first unikernel, you may use a read-only image plus an append-only data format. The xv6 filesystem phase still teaches why writes and snapshots require quiescing and flush semantics.

## Implementation method for the labs

For every MIT lab:

1. Read the named xv6 chapter and source files.
2. Draw the relevant path before editing code.
3. Write the invariants and failure cases.
4. Add tracing before changing behavior.
5. Implement the smallest increment.
6. Run provided tests and add one adversarial test.
7. Write a postmortem explaining any incorrect first design.

## Instrumentation to add

Create a small tracing facility with fixed-size events:

```c
struct trace_event {
    uint64 ticks;
    uint32 cpu;
    uint16 kind;
    uint16 flags;
    uint64 a;
    uint64 b;
};
```

Use it for:

- syscall entry/exit;
- page faults;
- context switches;
- sleep/wakeup;
- lock contention;
- block transactions;
- network interrupts.

Export traces after tests rather than printing from sensitive paths.

## Debugging playbook

### xv6 hangs after a lock change

Check lock order, sleeping while holding a spin lock, interrupts disabled too long, and missing wakeups. Use per-lock acquisition counters and owner information in debug builds.

### Copy-on-write fails only during exit

Check reference counts, page-table teardown, shared trampoline mappings, and whether a page is decremented more than once.

### Filesystem test passes until simulated crash

Check whether commit state reaches disk before dependent metadata, whether recovery replays only complete transactions, and whether buffer-cache writes escape the intended transaction.

## Exercises

1. Add a syscall trace filtered by process and syscall number.
2. Add page-fault counters by cause and render a histogram.
3. Implement a scheduling policy and measure fairness and turnaround.
4. Add lock-wait instrumentation and identify the hottest lock.
5. Inject a crash after each filesystem log write and classify recovery outcomes.
6. Write a document listing every xv6 subsystem your first unikernel omits and where its responsibilities move.

## Review questions

1. Why does a process need both a user stack and a kernel stack?
2. Which state is saved by the trap entry and which by the context switch?
3. What prevents a lost wake-up in xv6?
4. Why is copy-on-write fundamentally an ownership problem?
5. What guarantee does the filesystem log provide, and what does it not provide?
6. Which process responsibilities must Opencomputer still implement externally?

## Opencomputer connection

xv6 provides a useful responsibility map. Opencomputer replaces process isolation with VM isolation, replaces process lifecycle with sandbox lifecycle, replaces an in-guest package environment with immutable artifacts and attached volumes, and replaces local syscalls for orchestration with a host-control protocol. The abstractions move across the guest/VMM boundary; they do not vanish.
