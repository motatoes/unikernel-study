> Part of the [Unikernel Engineering Handbook](../README.md).

## Related handbook chapters

- [Chapter 17 — Security Engineering for a Unikernel Platform](../chapters/17-unikernel-security.md)
- [Chapter 14 — VMM Snapshots, Isolation, and Defensive Device Models](../chapters/14-vmm-snapshot-security.md)
- [Chapter 18 — Performance, Measurement, and Observability](../chapters/18-performance-observability.md)

---

# Week 59 — Security, fuzzing, failure injection, and snapshot safety

**Objective:** Treat the unikernel and snapshots as hostile-input surfaces rather than assuming small means secure.

### Reading

- [ ] *A Security Perspective on Unikernels*.
- [ ] *Assessing Unikernel Security*.
- [ ] Firecracker design and snapshot-security guidance.

### Implementation

- [ ] Enforce W^X.
- [ ] Keep null page unmapped.
- [ ] Add stack guard pages and stack protection where supported.
- [ ] Bound all packet, queue, control, and application parsers.
- [ ] Fuzz ELF, manifest, network, virtqueue, control, block-metadata, and snapshot parsers.
- [ ] Add device allowlisting.
- [ ] Add snapshot authentication and encryption at the platform layer.
- [ ] Reseed entropy after restore or clone.
- [ ] Rotate ephemeral credentials after restore where required.
- [ ] Inject packet loss, malformed descriptors, delayed interrupts, block errors, OOM, and control-channel failures.

Quiesce sequence:

```text
1. Stop accepting new work.
2. Drain or cancel in-flight application operations.
3. Flush block writes.
4. Drain or disable device queues.
5. Record application checkpoint metadata.
6. Report quiesced state.
7. Capture vCPU, memory, and VMM device state.
8. Resume or terminate.
```

Restore sequence:

```text
1. Validate snapshot format, image, configuration, CPU, and device topology.
2. Restore memory and device state.
3. Correct monotonic-time accounting.
4. Reseed entropy.
5. Rotate ephemeral credentials if required.
6. Re-establish host control.
7. Validate attached block identities.
8. Resume application work.
```

### Verification

- [ ] Fuzz targets run continuously or on every relevant change.
- [ ] Restored clones do not share RNG state.
- [ ] Wrong image/configuration/device topology is rejected.
- [ ] Threat model covers compromised guest and malicious virtual-device responses.

### Exit artifact

```text
docs/THREAT-MODEL.md
docs/SNAPSHOT-FORMAT.md
fuzz/
tests/failure-injection/
```

---
