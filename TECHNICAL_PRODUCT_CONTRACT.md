# Local-first Collaborative Workboard

## Technical Product Contract

**Version:** 0.1  
**Status:** FROZEN — Initial technical boundary  
**Authority:** Client technical acceptance contract  
**Frozen:** 2026-09-22  
**Relationship to Product Definition:** This document specializes the behaviour required by `PRODUCT_DEFINITION.md`. It does not prescribe Anvil execution methodology.

---

## 1. Technical objective

Build a software system in which multiple clients maintain local application state and exchange mutations with a durable server while preserving explicitly defined correctness properties under:

- temporary network loss;
- retries;
- duplicate delivery;
- stale reads;
- concurrent mutations;
- backend restart;
- client restart;
- synchronization delay.

The objective is not to implement every distributed-systems technique.

The objective is to implement the simplest architecture that can validly demonstrate the required Product semantics.

---

## 2. Required topology

The system must contain at least the following logical responsibilities.

### 2.1 Client application

Responsible for:

- UI;
- local persisted state;
- local mutation;
- pending-operation persistence;
- synchronization;
- conflict presentation;
- recovery of usable state after reconnect or local rebuild.

The exact client architecture and implementation technology are engineering decisions.

### 2.2 Server/API

Responsible for:

- shared authoritative state or equivalent durable canonical representation;
- validation;
- mutation acceptance/rejection;
- concurrency/version checks;
- idempotency;
- synchronization interfaces;
- durable persistence;
- recovery after normal process restart.

The exact protocol, framework and server architecture are engineering decisions.

### 2.3 Durable server persistence

The server must have a durable persistence mechanism capable of satisfying the confirmed-write, idempotency, recovery and transactional invariants in this contract.

The persistence technology is an engineering decision.

### 2.4 Local client persistence

The client must have browser-compatible durable local persistence sufficient to preserve:

- previously loaded supported state;
- supported offline work;
- pending operations;
- synchronization state;
- unresolved conflicts where required.

The persistence technology is an engineering decision.

---

## 3. Implementation stack

The implementation stack is an engineering decision.

No programming language, framework, runtime, database, client-side persistence technology, infrastructure component or development tool is mandated by this commission unless a later Product requirement or observed engineering evidence makes one necessary.

The stack must be selected under the Product requirements and this technical contract, prioritizing the simplest technically sufficient combination of:

- programming language;
- runtime;
- libraries and frameworks;
- client-side persistence;
- server-side persistence;
- development and packaging tooling;
- testing and validation tooling;
- operational tooling where materially required.

Selection should consider, proportionally to the need:

- problem and domain fit;
- existing ecosystem maturity;
- browser/platform support;
- synchronization correctness;
- interoperability;
- testability and reproducibility;
- maintainability;
- security and failure behaviour;
- operator/developer competence and learning cost;
- operational simplicity;
- integration, migration and maintenance burden;
- total lifecycle cost;
- reversibility and cost of later change.

An existing technology choice, once introduced, should be preserved when it remains technically adequate.

A different language, runtime, framework, persistence technology or other stack component should be introduced only when its expected marginal value justifies the additional complexity and lifecycle cost.

Uniformity is not an objective by itself.

Diversity is not an objective by itself.

Different coherent components may use different technologies when their requirements materially justify doing so.

Technology must not be introduced for novelty, prestige or merely to exercise available engineering or Anvil capabilities.

---

## 4. Core state model

Every synchronizable entity must have:

- stable logical identity;
- sufficient version/concurrency metadata;
- durable server representation;
- client representation.

Every synchronizable client mutation must have a stable operation identity.

A conceptual operation contains at least information equivalent to:

```text
operation_id
client_id
entity_id
entity_type
base_version / equivalent causal reference
operation_type
payload
created_at_local
```

The exact representation may differ.

The implementation must document:

- which information establishes logical identity;
- which information establishes operation identity;
- how concurrency is determined;
- how acknowledgement relates to durable server state.

Reusing an `operation_id` for materially different logical operation content is invalid. In particular, the same `operation_id` must not silently identify a different target, operation type, payload or relevant causal/base-version meaning. Such reuse must be rejected or surfaced as an operation-identity inconsistency; it must not be treated as a successful duplicate of the original operation.

---

## 5. Server acknowledgement semantics

The implementation must define precisely what acknowledgement means.

Minimum requirement:

Once the server reports a mutation as durably accepted, a normal process restart must not cause that accepted mutation to disappear.

The implementation must not acknowledge persistence before satisfying its own durability contract.

If acknowledgement has multiple levels or meanings, they must be explicitly distinguished.

---

## 6. Client mutation states

A local mutation should have an explicit lifecycle equivalent to:

```text
LOCAL_PENDING
    ↓
SENDING
    ↓
CONFIRMED
```

with possible branches such as:

```text
FAILED_RETRYABLE
CONFLICT
REJECTED
```

Exact naming is an implementation choice.

The UI need not expose internal state names but must make the meaningful user states intelligible.

A pending local change must not be presented as confirmed merely because it exists locally.

---

## 7. Required correctness invariants

### INV-01 — Confirmed-write durability

A server-confirmed mutation must survive a normal server process restart.

### INV-02 — Idempotent logical operation

Processing the same logical `operation_id` multiple times must not produce additional unintended domain effects.

### INV-03 — Persistent offline intent

Supported local mutations made offline must survive the supported client restart/reload boundary until:

- confirmed;
- resolved;
- rejected;
- or intentionally discarded by the user through supported Product behaviour.

### INV-04 — No silent conflicting overwrite

If two concurrent mutations are semantically incompatible under the domain merge rules, neither user intent may be silently discarded.

### INV-05 — Final convergence

When:

- all reachable pending operations have been processed;
- all material conflicts have been resolved;
- clients have synchronized;

then clients must observe equivalent confirmed domain state.

Equivalent does not require identical internal representation where the Product semantics do not require it.

### INV-06 — Causal/version validation

A mutation based on stale state must not be accepted as if it were based on current state unless the operation's explicit merge semantics permit it.

### INV-07 — Auditable material mutation

For a material shared-state change, the system must preserve enough information to explain:

- its origin;
- its relevant ordering/version context;
- and its accepted result.

### INV-08 — Conflict resolution is itself durable state

Resolving a conflict must produce a durable, identifiable state transition rather than merely changing local UI state.

### INV-09 — Rebuildability

A client must be capable of discarding its local confirmed cache and rebuilding valid confirmed state from authoritative server state.

### INV-10 — Failure visibility

A synchronization failure that prevents successful completion must not be represented as successful synchronization.

---

## 8. Conflict semantics

The project must explicitly define which supported operations:

- always conflict;
- can merge automatically;
- can be applied independently;
- require user judgment.

The initial implementation need only support a bounded, coherent ruleset.

For example, given task version 4:

Client A:

```text
status TODO -> DONE
```

Client B:

```text
title "Fix payment" -> "Fix payment retry"
```

These may be defined as compatible field-level changes.

By contrast:

Client A:

```text
priority LOW -> HIGH
```

Client B:

```text
priority LOW -> MEDIUM
```

must produce a conflict unless another explicit domain rule justifies an automatic result.

last-write-wins must not be used as a hidden universal conflict policy.

If last-write-wins or another automatic rule is used for a particular field or operation type, that behaviour must be explicit and justified.

---

## 9. Synchronization protocol requirements

The exact synchronization protocol is an engineering decision.

It must support behaviour equivalent to the following.

### Push

A client can send pending mutations.

### Acknowledgement

The server can identify the relevant result of an operation, including where applicable:

- accepted;
- duplicate/already processed;
- rejected;
- conflicting.

### Pull / catch-up

A client can obtain enough confirmed state or change information to become current relative to the server.

### Retry

Transient network failure can be retried safely.

### Stale-client catch-up

A client that has been disconnected for a significant period can retrieve enough newer state to become usable again.

### Duplicate delivery

Duplicate requests remain safe under the idempotency contract.

No specific transport mechanism is mandated.

Persistent connections are not required unless justified by Product or engineering evidence.

---

## 10. Ordering

The implementation must define its ordering guarantees.

It does not need to provide a total global ordering unless that is required by the chosen Product semantics.

Where ordering affects correctness, the system must preserve or detect the relevant ordering explicitly.

Documentation must not imply stronger ordering guarantees than the implementation actually provides.

---

## 11. Time

Wall-clock timestamps must not be treated as sufficient causal ordering for correctness.

Timestamps may be retained for:

- human-readable history;
- diagnostics;
- display;
- operational evidence.

Concurrency/version semantics must rely on appropriate explicit state, version or causal information.

---

## 12. Deletion

Deletion has difficult synchronization semantics.

The implementation must either choose and document a coherent supported approach.

### Option A — synchronized deletion

Support deletion with an explicit durable deletion/tombstone or equivalent contract capable of preserving stale-client and synchronization correctness.

### Option B — deletion outside initial synchronizable scope

Declare deletion outside the initial supported synchronizable Product scope.

Silent physical removal that breaks stale-client synchronization is not acceptable.

The engineering process may choose either option if Product acceptance remains satisfied.

---

## 13. Database and persistence transactions

Server mutations that require multiple related durable writes to satisfy one domain invariant should execute atomically or with another mechanism that provides equivalent correctness.

In particular, idempotency state and the corresponding accepted domain mutation must not be capable of obviously diverging through a partial-commit path.

The implementation must document the relevant transactional or atomicity boundary.

The specific persistence technology does not determine the acceptance result; the observable guarantees do.

---

## 14. Schema evolution

The project must include at least one demonstrated server-side schema or durable-data migration relevant to the actual chosen implementation.

Client/server compatibility must be considered if local persisted state can survive application upgrades.

The project does not need a complex compatibility framework.

It must, however, define what happens when an older supported local schema encounters a newer application/server version.

The behaviour may include:

- migration;
- controlled invalidation and rebuild;
- compatibility handling;
- explicit unsupported-version recovery.

Silent corruption or undefined failure is not acceptable.

Migration, controlled invalidation, rebuild or application upgrade must not silently discard supported pending operations or unresolved material conflicts. If pending intent cannot be migrated or applied under the newer state, the system must preserve enough information to surface an explicit supported reconciliation, rejection, unsupported-version recovery or user-directed discard path.

---

## 15. Security boundary

This is not an enterprise-security project.

Minimum expectations:

- external inputs are validated;
- no obvious injection vulnerability is knowingly left in principal supported paths;
- secrets are not committed;
- durable server persistence is not exposed unnecessarily;
- dependency management is documented;
- a client cannot arbitrarily declare authoritative server version/current state without validation;
- security limitations are documented honestly.

Authentication may be simplified.

If authentication is omitted or intentionally minimal, public documentation must state that clearly.

Do not claim production security unless separately demonstrated.

---

## 16. Observability

The system must provide enough technical visibility to diagnose synchronization behaviour.

At minimum, useful evidence should exist for materially relevant cases such as:

- synchronization attempts;
- accepted operations;
- duplicate operations;
- retries;
- failed operations;
- conflicts;
- conflict resolutions;
- relevant client/server version or synchronization cursor;
- synchronization latency where practical.

This may be implemented through:

- logs;
- structured diagnostic output;
- a small diagnostic surface;
- retained operation/history information;
- or another technically appropriate mechanism.

A production observability platform is not required.

Observability must remain proportionate to the Product.

---

## 17. Testing strategy

Testing is a major Product and technical requirement.

### 17.1 Unit tests

Cover deterministic domain behaviour, particularly:

- merge compatibility;
- conflict detection;
- idempotency decisions;
- state transitions;
- supported conflict-resolution rules.

### 17.2 Persistence / integration tests

Exercise materially relevant behaviour including:

- durable persistence;
- transaction/atomicity behaviour;
- duplicate operation handling;
- version/concurrency checking;
- restart-safe confirmed state.

### 17.3 Client synchronization tests

Exercise:

- pending operations;
- offline state;
- reconnect;
- retry;
- duplicate acknowledgement;
- stale state;
- successful synchronization;
- failed synchronization.

### 17.4 End-to-end tests

At least one automated or repeatably executable end-to-end scenario must exercise two independent clients.

The test must exercise real Product behaviour rather than merely calling isolated domain functions.

### 17.5 Failure / fault tests

The project must test selected failure situations such as:

- request lost before reaching server;
- response lost after server commit;
- duplicate retry;
- client reconnect after stale state;
- backend restart;
- concurrent compatible edits;
- concurrent incompatible edits.

Tests must preserve the distinction between simulated failure evidence and evidence obtained from real application boundaries.

---

## 18. Property / invariant testing

The project should include an automated simulation or property-oriented test harness if proportionate to the implementation.

A useful harness may generate or replay sequences containing actions such as:

```text
create
edit
disconnect
reconnect
retry
duplicate
concurrent edit
resolve conflict
restart server
sync
```

and validate the invariants defined in this contract.

The exact testing framework is not prescribed.

At minimum, automated testing must go beyond a small collection of scripted happy paths.

If a property-oriented harness proves unnecessarily costly relative to the implementation, an alternative systematic invariant-testing approach may be used if it provides equivalent useful evidence.

---

## 19. Demonstration fault simulator

For portfolio and review value, provide a reproducible way to demonstrate difficult synchronization behaviour without requiring ad hoc manual network manipulation.

This may be:

- development controls;
- a deterministic scenario runner;
- a test harness;
- a fault-injection layer;
- a proxy;
- another simple engineering mechanism.

The demonstration should be able to induce at least:

- offline client;
- delayed synchronization;
- duplicate submission;
- concurrent updates;
- conflict.

Do not build a general chaos-engineering platform.

---

## 20. Performance envelope

This is not a scale benchmark.

The product should remain responsive for a realistic demonstration workspace.

A modest target envelope should be defined and measured during implementation, sufficient to exercise:

- hundreds to low thousands of work items where practical;
- multiple browser clients;
- a meaningful backlog of pending operations after temporary disconnection;
- synchronization recovery after reconnect.

Exact limits must be measured rather than invented.

No claim of large-scale production performance is required.

Performance optimization must remain subordinate to correctness and Product usefulness unless evidence identifies a material bottleneck.

---

## 21. Recovery

Required recovery scenarios include:

### R1 — Backend restart

Confirmed server data survives a normal backend restart.

### R2 — Client confirmed-cache loss

A client can rebuild valid confirmed state from the server after discarding its local confirmed cache. This rebuild must not silently discard supported pending operations or unresolved material conflicts.

### R3 — Pending local work

Pending work is not silently classified as confirmed after a client restart.

### R4 — Invalid or stale pending operation

The system surfaces rejection, reconciliation or conflict rather than silently discarding the operation.

Recovery behaviour must be reproducible and documented.

---

## 22. Data export / debugging

For portfolio, recovery and technical understanding, it should be possible to inspect or export enough state to understand materially relevant information such as:

- current entities;
- pending operations;
- conflicts;
- operation history;
- version/concurrency information;
- resolution history.

This does not need to be a polished end-user feature.

Diagnostic mechanisms must not become a substitute for understandable Product behaviour.

---

## 23. Technical documentation

The public repository must include documentation sufficient for a technically competent reviewer to understand and reproduce the system.

### README.md

Include:

- Product purpose;
- setup/run instructions;
- quick demonstration;
- major limitations.

### ARCHITECTURE.md

Include:

- major components;
- client/server authority model;
- synchronization flow;
- persistence boundaries;
- material architectural trade-offs.

### SYNC_PROTOCOL.md

Include:

- operation identity;
- version/concurrency semantics;
- push/pull or equivalent synchronization behaviour;
- acknowledgement semantics;
- retries;
- duplicate handling;
- stale-client catch-up.

### CONFLICT_MODEL.md

Include:

- compatible changes;
- conflicting changes;
- automatic merge behaviour;
- explicit resolution behaviour;
- unsupported cases.

### INVARIANTS.md

Include:

- exact correctness properties;
- how each property is validated or tested;
- known limitations in the evidence.

### FAILURE_MODEL.md

Include:

- supported failure classes;
- expected behaviour;
- recovery behaviour;
- known unsupported failure classes.

### DECISIONS/

Maintain a small set of relevant architectural decision records for material decisions.

Do not create ADRs for trivial implementation choices.

---

## 24. Minimum required technical demonstrations

### DEMO-01 — Offline creation or modification

A client creates or changes supported work offline and successfully synchronizes later.

### DEMO-02 — Lost response / retry

The server processes an operation, the response is effectively lost, the client retries, and the domain effect occurs only once.

### DEMO-03 — Compatible concurrent modification

Two independent clients edit compatible fields and final confirmed state preserves both compatible intents.

### DEMO-04 — Material conflict

Two independent clients edit an incompatible field or property.

The conflict is detected and surfaced.

Neither material intent is silently discarded.

### DEMO-05 — Conflict resolution

The conflict is explicitly resolved and clients subsequently converge.

### DEMO-06 — Stale-client recovery

A client that has missed multiple changes catches up correctly and returns to a usable synchronized state.

### DEMO-07 — Backend restart

Confirmed state survives restart and synchronization continues correctly.

### DEMO-08 — Local rebuild

A client discards its local confirmed cache and reconstructs valid confirmed state from the server without silently losing supported pending intent or unresolved material conflicts.

---

## 25. Technical non-goals

Do not introduce the following unless justified by an actual Product or engineering requirement:

- CRDT framework;
- operational transformation;
- event sourcing;
- message broker;
- distributed log infrastructure;
- container orchestration platform;
- service mesh;
- microservices;
- distributed cache;
- GraphQL;
- persistent real-time transport;
- cloud provider-specific architecture;
- multi-region replication;
- consensus algorithm;
- custom distributed database.

These examples are not prohibited technologies.

They are explicitly non-mandatory.

They should be introduced only where their expected value is supported by a real requirement or observed evidence.

A comparatively simple architecture with a single application backend and one durable persistence layer is acceptable when it validly satisfies the Product and technical contract.

Infrastructure complexity is not evidence of engineering quality.

---

## 26. Evidence required for completion

The final technical acceptance package must contain:

- passing automated tests;
- reproducible end-to-end evidence;
- demonstrated failure cases;
- explicit invariant coverage;
- architecture documentation;
- synchronization and conflict documentation;
- known limitations;
- clean setup from a fresh checkout;
- reproducible principal demonstrations;
- a short public demonstration suitable for portfolio use.

A visually working application with untested synchronization semantics is not sufficient.

A sophisticated synchronization implementation without a usable Product is also not sufficient.

Product behaviour and technical correctness must close together.

Evidence must distinguish:

- demonstrated behaviour;
- inferred behaviour;
- intentionally unsupported behaviour;
- unresolved limitations.

---

## 27. Technical evolution rule

When implementation discovers that an initial technical assumption is invalid:

1. preserve the evidence;
2. identify the affected Product or technical requirement;
3. determine whether the required response is an engineering decision or a Product decision;
4. choose the simplest technically sufficient repair satisfying the existing Product contract where that repair is legitimately derivable;
5. do not broaden the project merely because a more sophisticated technical technique exists;
6. revalidate the affected invariants and acceptance evidence.

A materially different Product contract must return to the Human/DO.

Normal technical design decisions remain the responsibility of the executing engineering process.

---

## 28. Product ambiguity and clarification boundary

The client defines:

- Product intent;
- required behaviour;
- Product constraints;
- invariants;
- acceptance outcomes.

The executing engineering process owns derivable engineering decisions, including:

- architecture;
- implementation stack;
- technical decomposition;
- implementation approach;
- validation strategy;
- testing strategy;
- engineering repair.

A material unresolved Product ambiguity must not be silently converted into an engineering decision.

If a material ambiguity remains about:

- intended user behaviour;
- Product semantics;
- acceptance criteria;
- Product scope;
- Product trade-offs;
- what constitutes acceptable loss;
- conflict behaviour;
- recovery behaviour;
- failure behaviour;
- or another decision that would materially alter the Product outcome,

and the answer cannot be legitimately derived from `PRODUCT_DEFINITION.md`, this contract and observed Product evidence, the ambiguity must be surfaced to the Human/DO.

A clarification request should state:

1. what is unclear;
2. why the ambiguity matters;
3. the materially different interpretations;
4. what work, if any, can continue safely without the answer.

Ordinary reversible engineering decisions must not be escalated merely because several technically valid alternatives exist.

Uncertainty is not by itself a reason to ask the Human.

The relevant distinction is:

Product uncertainty that materially changes the intended outcome must be surfaced.

Engineering uncertainty should normally be investigated and resolved by engineering.

---

## Technical completion criterion

Technical completion requires a reproducible local-first system that:

- persists supported local work;
- safely synchronizes;
- handles retries idempotently;
- detects meaningful concurrent conflicts;
- merges at least one useful class of independent concurrent changes;
- supports explicit conflict resolution;
- preserves confirmed durable state;
- survives the defined restart and recovery scenarios;
- supports stale-client catch-up;
- supports local-state rebuild;
- converges after synchronization and conflict resolution;
- exposes synchronization state sufficiently for users and reviewers to understand what is happening;
- demonstrates those claims through automated, failure-oriented and end-to-end evidence.

Technical completion alone does not override the Product completion criterion in `PRODUCT_DEFINITION.md`.

The commission is complete only when the Product behaviour and the technical evidence satisfy both documents together.
