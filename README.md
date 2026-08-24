# omnux-gpu

An attempt at an MIT-licensed GPU driver for Apple M3-series silicon.
License: MIT. Upstream may take everything; that is the point.

## The honest state of this project

This repository contains project scaffolding, a research roadmap, tooling,
and eventually driver code. It does **not** yet contain a working driver,
and nothing here will claim to until pixels appear on a physical M3 machine.

Writing a working M3 GPU driver requires reverse-engineering:

1. **Command processor submission model** — how macOS userspace submits
   work to the AGX firmware on T8122/T603x (differs from M2).
2. **Shader ISA deltas** — M3 introduced Dynamic Caching, mesh shaders and
   hardware ray tracing; the instruction encoding changed vs M1/M2.
3. **Firmware interface** — version negotiation, queue management, fault
   handling against shipping macOS firmware.

Each of these is only discoverable **against real hardware**, using m1n1's
proxyclient tracing on a machine booted into macOS with instrumentation.
No amount of code written away from the metal can substitute for that loop.

## Repository layout

    docs/       ISA notes, capture methodology, findings as they land
    tools/      trace post-processing, ISA diffing utilities
    src/        driver sources (kernel-side; license-clean, clean-room)

## Method

1. Capture: boot target M3 Mac into m1n1 proxyclient harness, run macOS
   GPU workloads under trace (`m1n1/proxyclient/experiments/agx_*.py`).
2. Diff: compare captures against known M1/M2 models; document deltas in `docs/`.
3. Implement: clean-room driver code in `src/` from documented behavior only.
4. Validate: boot Omnux kernel with the driver via kexec from m1n1; iterate
   until DRM render nodes appear and glmark runs.
5. Publish: every milestone goes upstream to Asahi first.

## Hardware access

The critical path is an M3-class Mac (any) plus a second machine for the
proxyclient USB session. If you have hardware to lend, open an issue —
this is the bottleneck, full stop.
