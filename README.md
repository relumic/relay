# Relay
Priority-ordered data sync for unreliable networks

> **Status:** Early development. No code released yet.

---

## The Problem

In defense, industrial, and edge environments, connectivity isn't reliable. When a connection window opens, a naive queue delivers data in arrival order - bulk transfers consume bandwidth while high-priority payloads wait.

Relay fixes that: **highest-priority data moves first, always.**

---

## Planned Capabilities

- **Guaranteed delivery.** Data is persisted to disk immediately - not lost on crash or dropped connection.
- **Priority-ordered sync.** Highest-priority payload transmits first within any connection window.
- **Resilient by default.** Resumes exactly where it left off after disconnection or restart.
- **Zero runtime dependencies.** Single binary. Runs anywhere - cloud, edge, air-gapped hardware.

---

## Roadmap

- **V1 - Core Sync Engine** *(in development)*: Priority queue, WAL persistence, gRPC transport, acknowledgment protocol, conflict resolution, single binary, Docker image, chaos test suite.
- **V2 - Visibility Layer**: Web dashboard, metrics, configurable retry/timeout policies.
- **V3 - Policy Engine**: Rule-based priority config, data TTL, bandwidth budgeting, audit log.
- **V4 - Edge AI**: On-device content-aware prioritization.

---

## About

Built by [Relumic](https://relumic.com) - edge computing and open systems software for defense, government, and critical infrastructure.

[contact@relumic.com](mailto:contact@relumic.com) | [relumic.com](https://relumic.com)

---

## License

Apache 2.0
