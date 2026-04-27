---
title: "Designing Distributed Systems: 8 Transactional Saga Patterns"
classes: wide
---

## Coupling

**Coupling** is the degree of interdependence between software modules and components. There are two types: **static** and **dynamic**.

### Static Coupling

Fixed, code/deployment-level dependencies. Example: a shared library, or direct code references — regardless of whether services are actively communicating.

### Dynamic Coupling

Runtime dependencies. Example: when Service A calls Service B synchronously, A becomes temporarily coupled to B's:

- **Availability** — if B is down, A fails too
- **Performance** — if B is slow, A is slow
- **Error behaviour** — B's errors propagate directly to A

---

## The Three Primal Dynamic-Coupling Forces

Every distributed system is shaped by three fundamental tensions:

| Force | Options |
|---|---|
| **Communication** | Synchronous ↔ Asynchronous |
| **Consistency** | Atomic ↔ Eventual |
| **Coordination** | Orchestration ↔ Choreography |

These three axes combine to form the eight saga patterns.

---

## 1. Communication

### Synchronous vs. Asynchronous

- **Blocking (sync):** The caller waits. REST, SOAP, RPC (including gRPC).
- **Non-blocking (async):** The caller does not wait. Request/reply messaging, async REST, streaming, queues, topics, WebSockets.

#### Performance Impact

A concrete example using a trade rejection workflow:

| Approach | Total Latency |
|---|---|
| Synchronous (waiting at each step) | ~2,700 ms |
| Asynchronous (concurrent calls) | ~425 ms |

Async communication can deliver a **~6× performance improvement** in multi-step workflows.

#### The Architectural Quantum

> *An architecture quantum establishes the scope for a set of architectural characteristics.*

Characteristics of a quantum:
- Independent deployment
- High functional cohesion
- Low external-implementation static coupling
- Synchronous communication with other quanta

**Key insight:** Synchronous calls create *"dynamic quantum entanglement"* — they couple the operational characteristics of separate services. A slow or unavailable downstream service degrades the entire call chain.

#### Trade-off Summary

| | Synchronous | Asynchronous |
|---|---|---|
| Easy to reason about | ✅ | |
| Mimics non-distributed calls | ✅ | |
| Easier to implement | ✅ | |
| Highly decoupled | | ✅ |
| High performance and scale | | ✅ |
| Common performance tuning technique | | ✅ |
| Performance impact on interactive systems | ⚠️ | |
| Creates dynamic quantum entanglements | ⚠️ | |
| Complex to build and debug | | ⚠️ |
| Difficult for transactional behaviour | | ⚠️ |
| Complex error handling | | ⚠️ |

---

### Managing Contracts

#### Strict vs. Loose Contracts

```
Strict ←————————————————————————————→ Loose
 XML Schema  JSON Schema  GraphQL  Simple JSON  KVP arrays
 RPC/gRPC                Object
```

| | Tight (Strict) | Loose |
|---|---|---|
| Guaranteed contract fidelity | ✅ | |
| Build-time validation | ✅ | |
| Better for decoupled architectures | | ✅ |
| Easier to evolve | | ✅ |
| Decouples integration from implementation platform | | ✅ |
| Brittle integration points | ⚠️ | |
| Requires versioning | ⚠️ | |
| Less certainty; needs fitness functions | | ⚠️ |
| Requires developer discipline | | ⚠️ |

**Principle:** *Transfer values, not types.* Value-based (loose) contracts decouple consumers from implementation changes.

#### Contract Fitness Functions

Consumer-Driven Contracts (CDCs) can automate contract verification. The **Pact** framework ([docs.pact.io](https://docs.pact.io)) supports this for REST and event-driven architectures, with CI/CD integration.

---

### Event Payload Design

A key design question: should an event carry the **full data payload** or just **key identifiers**?

| Trade-off Dimension | Full Payload | Key-Only Payload |
|---|---|---|
| Scalability & performance | ⚠️ Poor | ✅ Good |
| Contract management & versioning | ⚠️ Complex | ✅ Simple |
| Single system of record | ⚠️ Multiple | ✅ Single |
| Stamp coupling & bandwidth | ⚠️ High | ✅ Low |

**Stamp coupling example:** A customer profile returning 500 KB payloads at 2,000 req/s uses ~1,000,000 KB/s of bandwidth. Returning only required fields (~200 bytes) reduces this to ~400 KB/s — a **2,500× reduction**.

> **Looser contracts create less brittle software architectures.**

---

## 2. Coordination

### Orchestration vs. Choreography

| | Orchestration | Choreography |
|---|---|---|
| State owner | Central orchestrator | Distributed across services |
| Workflow control | Centralised | Emergent / event-driven |
| Error handling | ✅ Easier — one place | ⚠️ Harder — spread across services |
| Responsiveness | Moderate | ✅ High |
| Scalability / throughput | Moderate | ✅ High |
| Fault tolerance | ✅ Good | ✅ Better |
| Recoverability | ✅ Good | ⚠️ Difficult |
| Workflow / state management | ✅ Good | ⚠️ Complex |

- **Orchestration** — generally one orchestrator per major workflow; it owns state and communicates update points to services.
- **Choreography** — services react to events; no central controller; the workflow emerges from the chain of events.
- **Hybrids** — an orchestrated outer workflow can delegate to choreographed sub-workflows.

---

## 3. Consistency

### ACID Transactions

**Atomicity, Consistency, Isolation, Durability.**

In a distributed context, standard ACID properties break down:

| Property | Distributed Problem |
|---|---|
| **Atomicity** | Each service commits/rolls back independently — a failure mid-chain leaves partial state |
| **Consistency** | An error in one service (e.g. inventory) causes data inconsistency across others |
| **Isolation** | Inserted data may be visible to other services before the overall transaction completes |
| **Durability** | Data is only made permanent at the service level, not the transaction level |

### BASE Transactions

As an alternative, **BASE** (Basically Available, Soft state, Eventually consistent) is the natural fit for distributed, decoupled systems.

### Eventual Consistency Patterns

Three patterns for propagating state changes (e.g. deleting a customer across multiple services):

1. **Background Synchronisation** — a background job reads across databases to sync state.  
   ⚠️ Creates direct database coupling between services.

2. **Event-Based Data Synchronisation** — the originating service fires an event; consumers react.  
   ✅ Decoupled. Standard EDA pattern.

3. **Workflow Event Pattern** — a dedicated workflow processor mediates between the event producer and consumers, handling ordering and retries.

---

### Compensating Updates

When a step fails partway through a distributed transaction, previously committed changes must be **compensated** (rolled back manually).

**Fallacies of compensating updates:**
- The compensating update itself may fail
- Side effects may have already occurred (e.g. an email sent, a charge processed)
- State management becomes complex

**Mitigation:** Marking data as `WIP` vs `PLACED` allows services to avoid querying data not yet in a finalisable state, reducing inconsistency windows.

---

## 4. Transactional Sagas

The eight saga patterns are formed by combining the three coupling forces:

| Saga | Consistency | Coordination | Communication |
|---|---|---|---|
| **Epic Saga** | Atomic | Orchestration | Sync |
| **Fantasy Fiction Saga** | Atomic | Orchestration | Async |
| **Fairy Tale Saga** | Eventual | Orchestration | Sync |
| **Parallel Saga** | Eventual | Orchestration | Async |
| **Phone Tag Saga** | Atomic | Choreography | Sync |
| **Horror Story Saga** | Atomic | Choreography | Async |
| **Time Travel Saga** | Eventual | Choreography | Sync |
| **Anthology Saga** | Eventual | Choreography | Async |

---

### Saga Quality Profiles

Rating scale: `★★★` strong · `★★` moderate · `★` weak · ` ` poor

| Saga | Decoupled | Simple | Responsive | Scalable |
|---|---|---|---|---|
| Epic Saga | | ★★★ | | |
| Fantasy Fiction | | ★★ | ★★ | |
| Fairy Tale | | ★★★ | | |
| Parallel | | ★★ | ★★ | ★★ |
| Phone Tag | | ★ | | |
| Horror Story | ★ | | ★ | |
| Time Travel | ★★ | ★ | | |
| Anthology | ★★★ | | ★★ | ★★★ |

---

### Saga Descriptions

#### 🗡️ Epic Saga — *"A long-running, heroic story"*

- Mimics a non-distributed transactional interaction
- Holistic transactional coordination increases coupling
- Easy to understand; difficult to implement

**Use when:** Each step must complete before the next starts; absolute transactionality matters more than responsiveness.

---

#### 📖 Fantasy Fiction Saga — *"A complex story that's hard to believe in the end"*

- Moving to async communication improves performance…
- …but introduces concurrency issues that may be worse than the original performance problems

**Use when:** A first attempt at improving an Epic Saga; responsiveness matters more than strict ordering.

---

#### 🏡 Fairy Tale Saga — *"An easy story with a pleasant ending"*

- Sync + orchestrated = easiest to reason about
- The orchestrated saga in Richardson's patterns book

**Use when:** Medium to complex workflows that don't need extreme scale. **Default choice for most situations.**

---

#### ⚡ Parallel Saga — *"Multiple stories running at the same time"*

- Orchestrator allows complex workflows with concurrency
- Highly attractive for complex workflows at high scale
- Difficult if ordering of updates matters

---

#### 📞 Phone Tag Saga — *"Like the game of phone tag"*

- An unusual combination: atomic + choreography
- Adds scalability to a simple transactional workflow when the orchestrator becomes a bottleneck
- Not common in practice

---

#### 😱 Horror Story Saga — *"A nightmare"*

- Attempts atomic workflows without a coordinator, with concurrency on top
- Workflow, error handling, boundary conditions, and transactionality are spread across domain services
- Middling performance coupled with impossible-to-reproduce errors

⚠️ **Not uncommon in practice.** Usually a well-intentioned but flawed attempt to achieve high performance with atomicity.

---

#### ⏳ Time Travel Saga — *"A problem that moves atomically through time"*

- No orchestrator makes complex workflows difficult
- Best for pipeline problems (chain-of-responsibility, pipes-and-filters) with staging or additive workflows
- Works best when synchronous communication is acceptable

---

#### 📚 Anthology Saga — *"A loosely associated group of short stories"*

- Richardson's choreographed saga
- Polar opposite of the Epic Saga
- Best for non-transactional pipes-and-filters architecture styles
- Highly scalable due to lack of coupling

---

### Quantitative Trade-off Summary

| Axis | Impact on Decoupling / Scalability |
|---|---|
| Atomic → Eventual consistency | **+163%** improvement |
| Orchestration → Choreography | **+143%** improvement |
| Sync → Async communication | **+55%** improvement |

**Consistency choice** has the greatest impact on saga quality, followed by coordination style, then communication mode.

---

### Transactional Sharding

For very high-volume scenarios, **domain sharding** distributes load across multiple saga instances. Example: a concert ticket sale spike can be handled by sharding by seat section or venue zone, avoiding a single transaction bottleneck.

Domain sharding maps naturally onto microservices: services can be sharded by geography or specialisation, allowing concurrent saga instances without contention.

---

## Key Principles Summary

| Principle | Implication |
|---|---|
| Synchronous calls create dynamic quantum entanglement | Default to async where responsiveness and scale matter |
| Looser contracts = less brittle software | Prefer value-based contracts in microservices |
| Stamp coupling wastes bandwidth | Send only the data a consumer needs |
| ACID breaks in distributed systems | Accept eventual consistency where business rules allow |
| Compensating updates have fallacies | Use state management to reduce inconsistency windows |
| The Fairy Tale Saga is the default | Use it for most medium-complexity workflows |
| Horror Story is an antipattern | Avoid atomic + choreography + async |
| Anthology is the most scalable | Use for pipelines that don't require transactionality |
