> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 10 — Networking From Ethernet to an HTTP Service](../chapters/10-networking.md)
- [Chapter 18 — Performance, Measurement, and Observability](../chapters/18-performance-observability.md)

---

# Week 55 — Production HTTP unikernel

**Objective:** Produce the first useful, bounded application-specific image.

### Reading

- [ ] Revisit virtio-net and smoltcp integration.
- [ ] Review HTTP parser/library security assumptions.

### Implementation

- [ ] Integrate virtio-net and smoltcp into `oc-uk`.
- [ ] Implement a native `UnikernelApp` interface.
- [ ] Build `http-service` with health, version, metrics, and echo endpoints.
- [ ] Add bounded queues, request size, connections, and timeouts.
- [ ] Add readiness and graceful shutdown.
- [ ] Emit build, image, and configuration digests in logs.

Suggested application interface:

```rust
pub trait UnikernelApp {
    fn init(config: &AppConfig) -> Result<Self, AppError>
    where
        Self: Sized;

    fn run(&mut self, runtime: &Runtime) -> Result<(), AppError>;

    fn quiesce(&mut self, reason: QuiesceReason) -> Result<(), AppError>;

    fn shutdown(&mut self, reason: ShutdownReason);
}
```

### Verification

- [ ] Image boots and becomes ready under QEMU.
- [ ] Sustained load does not cause unbounded memory growth.
- [ ] Slow-client and malformed-request tests pass.
- [ ] Application shuts down cleanly through host control.

### Exit artifact

```text
oc-uk/apps/http-service/
docs/APPLICATION-ABI.md
```

---
