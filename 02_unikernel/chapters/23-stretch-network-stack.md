# Stretch Chapter 23 — Build a TCP/IP Stack From First Principles

## Goal

The core curriculum implements Ethernet, ARP, IPv4, ICMP, and UDP before integrating smoltcp. This stretch track continues into TCP and a more complete network subsystem. It is educational; a new stack should not replace a mature stack in production without extensive validation.

## 10-week sequence

### Week 73 — Packet-buffer and interface architecture

- Fixed and chained packet buffers.
- Checksummed cursor/parser API.
- Interface input/output queues.
- MTU and headroom handling.
- Deterministic packet test vectors.

### Week 74 — IPv4 fragmentation and reassembly

- Fragment identification and offsets.
- Bounded reassembly table.
- Timeouts and overlap policy.
- Resource-exhaustion defenses.

You may choose to remain “reject fragments” in production; implementing reassembly is still educational.

### Week 75 — Routing and neighbor discovery

- Route table and longest-prefix match.
- ARP state machine and pending packets.
- Gateway handling.
- Bounded cache and retries.

### Weeks 76–78 — TCP state machine

Implement:

```text
LISTEN
SYN-SENT
SYN-RECEIVED
ESTABLISHED
FIN-WAIT-1/2
CLOSE-WAIT
CLOSING
LAST-ACK
TIME-WAIT
CLOSED
```

Add sequence-space comparisons that handle wrap-around, retransmission queues, acknowledgments, and orderly/reset close.

### Week 79 — Flow and congestion control

- Receive and send windows.
- Retransmission timeout estimation.
- Slow start and congestion avoidance experiment.
- Duplicate acknowledgments and fast retransmit.
- Bounded outstanding data.

### Week 80 — Socket API and executor integration

- Listener backlog.
- Connection handles with generations.
- Async read/write readiness.
- Cancellation and timeout.
- Half-close semantics.

### Week 81 — DNS client and configuration

- UDP DNS query/response parser.
- Bounded names and compression-pointer loop protection.
- Cache and timeout.
- Static versus platform-provided DNS configuration.

### Week 82 — Hardening and differential tests

- Packet corpus and fuzzing.
- Compare selected behavior with Linux or smoltcp.
- Run loss/reordering/duplication tests.
- Test connection and memory exhaustion.

## Correctness principles

- Sequence arithmetic uses wrapping comparisons, not ordinary integer ordering.
- Every connection owns bounded queues and timers.
- A packet cannot allocate unbounded state.
- Invalid segments elicit a defined ignore/reset policy.
- Timers use monotonic time.
- External TCP connections are not assumed portable across durable restore.

## Useful test tools

- packet-capture comparison;
- Scapy-generated packets;
- network namespaces;
- `tc netem` for loss, delay, duplication, corruption, and reordering;
- protocol fuzzers;
- reference clients/servers;
- packetdrill-style scripted interactions.

## Comparison report

Compare your stack with smoltcp on:

```text
code size
memory per connection
throughput
p99 latency
packet copies
API ergonomics
protocol coverage
fuzz results
failure behavior
```

The likely final architecture is to retain your small stack as a learning/reference implementation and use smoltcp or another maintained stack for production.

## Acceptance criteria

- HTTP works over your TCP implementation under clean conditions.
- Retransmission works under controlled loss.
- Sequence wrap-around tests pass.
- Every connection and global table is bounded.
- Parser and state machine survive fuzzing.
- The report clearly states unsupported TCP/RFC behavior.
