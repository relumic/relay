# Relay Architecture

> **Status:** Design phase. No code written yet.

---

## Core Problem

In DIL (Disconnected, Intermittent, Limited-bandwidth) environments, connectivity loss is the normal operating mode. When a connection window opens, a naive queue delivers in arrival order - bulk transfers block high-priority payloads.

**Relay's guarantee: highest-priority data moves first, always.**

---

## V1 Components

### Write-Ahead Log (WAL)
Every payload is written to disk before any other operation. On crash recovery, the WAL is replayed to restore queue state.

**Implementation:** SQLite in WAL journal mode, `PRAGMA synchronous = FULL`.

### Priority Queue
Payloads ordered by: (1) priority level (1–5, 1 = highest), then (2) age (FIFO within level to prevent starvation).

**Implementation:** SQLite table with composite index on `(priority ASC, created_at ASC)`. Items remain in queue until the remote node sends an explicit acknowledgment.

### Sync Manager
Manages the connection lifecycle: monitors connectivity, drains the queue in priority order, handles disconnection, and resumes partial transfers on reconnect.

**Transport:** gRPC. Each payload has a unique ID assigned at WAL write time; the Sync Manager resumes from the last acknowledged ID on reconnect.

### Conflict Resolver
Last-write-wins using nanosecond timestamps assigned at `Send()` call time on the originating node. Tiebreak: higher priority wins; if equal, deterministic by ID.

### Client Interface
Three public methods. Nothing else is exported.

```go
Send(data []byte, priority int) error
Receive() ([]byte, error)
Status() (Status, error)
```

---

## Repository Structure

```
relay/
├── cmd/relay/main.go
├── internal/
│   ├── queue/
│   ├── wal/
│   ├── sync/
│   └── conflict/
├── pkg/client/       # Only public API surface
├── proto/relay.proto
└── docs/architecture.md
```

---

## Key Technology Decisions

| Concern | Choice | Status |
|---|---|---|
| Language | Go (preferred) | Undecided - Go vs. Rust, needs final call |
| Storage | SQLite | Strong preference |
| Transport | gRPC | Preferred; HTTP/2 overhead at ≤10 Kbps needs validation |
| Container | Docker / OCI | OCI required for air-gapped podman environments |

---

## Open Questions

2. **gRPC at low bandwidth:** Is HTTP/2 acceptable at ≤10 Kbps?
3. **Connectivity probe:** TCP SYN? gRPC health check?
5. **Queue depth limits:** Cap enforced? Eviction behavior?
6. **Clock skew:** What behavior is guaranteed when nodes have unsynchronized clocks?
7. **Conflict keys:** How are caller-supplied keys namespaced? What if none is supplied?
