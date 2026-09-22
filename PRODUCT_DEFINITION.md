# Local-first Collaborative Workboard

## Product Definition

**Version:** 0.1  
**Status:** FROZEN — Initial Product Authority  
**Authority:** Client Product Definition  
**Frozen:** 2026-09-22  
**Relationship to technical acceptance:** `TECHNICAL_PRODUCT_CONTRACT.md` specializes the technical behaviour and evidence required to satisfy this Product Definition. It does not replace Product Truth.

---

## 1. Product intent

Build a working, public, portfolio-ready collaborative workboard whose defining Product property is that useful work can continue locally during temporary network loss and synchronize later without silently losing user intent.

The Product must be useful as a workboard in its own right.

It is not a distributed-systems demonstration disguised as a Product.

Engineering sophistication is valuable only where it is required to deliver understandable, reliable Product behaviour.

---

## 2. Why this Product exists

Many useful applications must remain usable when connectivity is intermittent, delayed or unreliable.

For a collaborative workboard, the Product problem is not merely storing tasks.

The Product problem includes allowing people to:

- view useful previously available work while temporarily disconnected;
- make supported changes locally;
- understand whether a local change is pending, confirmed, conflicting or failed;
- reconnect and synchronize safely;
- collaborate from independent clients;
- preserve compatible concurrent intent where supported;
- surface incompatible concurrent intent rather than silently overwriting it;
- explicitly resolve material conflicts;
- recover after stale state, restart or local-state rebuild;
- understand enough system state to trust what the Product is telling them.

The resulting project must also be strong professional evidence of software Product judgment, architecture, implementation, testing, failure analysis and technical communication.

Portfolio value does not lower the Product correctness bar.

---

## 3. Intended users

The initial Product is for a small number of collaborators working on a shared workboard from independent client instances.

A user should not need distributed-systems knowledge to understand the meaningful state of their work.

The Product should make ordinary questions intelligible, including:

- Is this change saved locally?
- Has this change been confirmed by the shared system?
- Is synchronization still pending?
- Did synchronization fail?
- Is there a conflict requiring attention?
- Has the conflict been resolved?
- Am I looking at current confirmed state or stale/local state?

The initial Product does not require enterprise administration or large-scale collaboration.

---

## 4. Core Product behaviour

The Product must provide a usable shared workboard in which users can perform a bounded but coherent set of ordinary work-item operations.

At minimum, the Product must support behaviour sufficient to:

- create and view work items;
- modify supported work-item properties;
- organize or progress work through a meaningful workboard workflow;
- persist supported local work;
- continue supported work while offline;
- synchronize after reconnect;
- distinguish local pending intent from confirmed shared state;
- preserve safe retry behaviour;
- catch up a stale client;
- handle at least one useful class of compatible concurrent change;
- detect at least one material class of incompatible concurrent change;
- present material conflicts intelligibly;
- resolve a conflict explicitly;
- converge after pending work is processed and material conflicts are resolved;
- recover confirmed state after normal backend restart;
- rebuild confirmed client state from the server after local confirmed-cache loss.

The exact work-item fields, workflow labels, interaction design and internal representation are engineering decisions unless a later Product decision makes one material.

The supported domain must nevertheless remain recognizably useful as a collaborative workboard rather than a synthetic state machine built only to exercise synchronization.

---

## 5. Product states visible to the user

The implementation may use any internal state model that satisfies the Product and technical contracts.

The user experience must nevertheless preserve the meaningful distinction between states equivalent to:

- local work that exists but is not yet confirmed remotely;
- synchronization in progress;
- confirmed shared state;
- retryable synchronization failure;
- rejected work;
- material conflict;
- resolved conflict.

Internal names do not need to appear in the UI.

The UI must not imply successful confirmation merely because a change exists locally.

A synchronization failure must not be represented as successful synchronization.

---

## 6. Product invariants

### P-01 — No silent loss of supported user intent

Supported local or concurrent user intent must not disappear silently.

If the system cannot apply an intent, it must preserve enough information to surface rejection, failure, conflict or another explicit supported outcome.

### P-02 — Honest confirmation

The Product must distinguish locally persisted intent from server-confirmed shared state.

### P-03 — Persistent offline work

Supported offline work must survive the supported client restart/reload boundary until it is:

- confirmed;
- resolved;
- rejected;
- or intentionally discarded by the user through supported Product behaviour.

### P-04 — Safe retry

A transient communication failure must not cause the same logical user operation to produce unintended duplicate domain effects when retried.

### P-05 — No silent conflicting overwrite

When concurrent user intents are materially incompatible under the supported Product semantics, neither intent may be silently discarded as though no conflict occurred.

### P-06 — Useful compatible collaboration

The Product must support at least one useful class of concurrent change in which independent compatible user intents can both survive.

The exact merge mechanism is an engineering decision.

### P-07 — Explicit material conflict resolution

A material conflict must be surfaced and resolved through explicit supported Product behaviour.

Resolution must become shared durable state rather than only a local visual choice.

### P-08 — Eventual confirmed convergence

After reachable pending operations have been processed, material conflicts have been resolved and clients have synchronized, users must observe equivalent confirmed Product state.

### P-09 — Recoverability

Confirmed Product state must survive the defined backend restart boundary.

A client must be able to rebuild valid confirmed state from authoritative server state after loss of its local confirmed cache.

### P-10 — Pending intent survives maintenance boundaries

Client rebuild, cache invalidation, migration or application upgrade must not silently erase supported pending intent or unresolved material conflicts.

If existing pending intent cannot be migrated or applied, the Product must preserve and surface an explicit supported recovery, reconciliation, rejection or user-discard path.

### P-11 — Failure visibility

Material synchronization or recovery failure must be visible enough for the user to understand that successful completion has not occurred.

---

## 7. Offline and reconnect semantics

Offline operation is a first-class Product behaviour, not merely a failure fallback.

While disconnected, a supported client must be able to retain previously loaded usable state and make the supported local changes defined by the implementation.

Reconnect must not require the user to re-enter supported local work merely because the network was unavailable.

The Product may limit which operations are supported offline, but those limits must be explicit and coherent.

A temporarily disconnected client may become stale.

On reconnect, stale state must be reconciled through supported synchronization behaviour rather than silently being treated as current.

---

## 8. Collaboration and conflict semantics

The Product does not require every possible concurrent edit to merge automatically.

The supported rules must distinguish, coherently:

- changes that can coexist;
- changes that can merge automatically;
- changes that are materially incompatible;
- changes requiring user judgment.

Automatic behaviour must be understandable and documented.

A hidden universal last-write-wins policy that can silently discard material intent is not acceptable.

The exact merge architecture and concurrency mechanism remain engineering decisions.

---

## 9. Recovery and evolution

The Product must preserve understandable behaviour across the recovery cases defined by the Technical Product Contract, including:

- backend restart;
- stale-client catch-up;
- client confirmed-cache loss and rebuild;
- pending local work after client restart;
- invalid or stale pending work;
- supported local schema evolution.

A recovery operation must not manufacture confirmation that did not occur.

Schema evolution may use migration, controlled invalidation and rebuild, compatibility handling or another technically sufficient strategy.

Silent corruption, silent pending-intent loss or undefined success states are not acceptable.

---

## 10. Product usefulness and interaction quality

Technical correctness alone is insufficient if the resulting application is not useful or understandable as a workboard.

The Product must provide a user interface through which a reviewer can perform and understand the principal supported workflows.

The interface need not be visually elaborate.

It should make the following especially clear:

- workboard state;
- locally pending work;
- synchronization progress or failure;
- material conflicts;
- conflict resolution;
- return to synchronized confirmed state.

Diagnostic surfaces may supplement the Product UI but must not substitute for understandable Product behaviour.

---

## 11. Portfolio objective

The final public repository should allow a technically competent reviewer to understand:

- the Product problem;
- the intended users;
- the Product semantics;
- the major architecture;
- local and server persistence boundaries;
- synchronization behaviour;
- concurrency and conflict behaviour;
- relevant implementation decisions;
- invariant and failure testing;
- demonstrated recovery behaviour;
- limitations and unsupported cases;
- the evidence behind the principal claims.

A short reproducible working demonstration must be possible.

The portfolio package must distinguish demonstrated behaviour from inferred, unsupported or future behaviour.

---

## 12. Product boundaries and non-goals

The initial Product is not required to demonstrate:

- enterprise-scale multi-tenancy;
- enterprise RBAC or SSO;
- native mobile applications;
- large-scale or multi-region operation;
- high availability;
- real-time presence indicators;
- a broad project-management feature suite;
- production SaaS operations;
- production security certification;
- custom distributed infrastructure;
- every possible merge or conflict policy.

These are not permanently prohibited capabilities.

They enter scope only if they become necessary to satisfy the Product objective or are added by legitimate Product authority.

Complexity is not a Product objective.

---

## 13. Technical freedom

No particular programming language, framework, database, local persistence technology, synchronization algorithm or deployment architecture is part of Product Truth.

Engineering owns those choices within:

- this Product Definition;
- `TECHNICAL_PRODUCT_CONTRACT.md`;
- applicable governance and authority;
- observed technical evidence.

The Product requires the resulting behaviour and evidence, not a particular implementation technique.

---

## 14. Product acceptance

The initial Product is accepted only when the Product behaviour and the Technical Product Contract close together.

At minimum, acceptance requires demonstrated behaviour covering:

### AC-01 — Usable workboard

A user can create, view, modify and meaningfully organize supported work items through a usable UI.

### AC-02 — Offline work

Supported work performed offline persists locally and remains available across the supported client restart/reload boundary.

### AC-03 — Safe reconnect

Offline work can synchronize after reconnect without silent loss or unintended duplicate domain effect.

### AC-04 — Honest synchronization state

The user can distinguish meaningful pending, confirmed, failed and conflicting states.

### AC-05 — Compatible concurrent intent

Two independent clients can make at least one useful class of compatible concurrent change and final confirmed state preserves both compatible intents.

### AC-06 — Material conflict

Two independent clients can create at least one material incompatible concurrent change; the conflict is detected and neither intent is silently discarded.

### AC-07 — Durable conflict resolution

A user can explicitly resolve the conflict and clients subsequently converge on equivalent confirmed Product state.

### AC-08 — Stale-client recovery

A client that has missed multiple shared changes can catch up and return to a usable synchronized state.

### AC-09 — Backend restart

Confirmed shared state survives the defined normal backend restart boundary and synchronization remains usable afterwards.

### AC-10 — Local rebuild

A client can discard its local confirmed cache and reconstruct valid confirmed state from the server without silently losing supported pending intent.

### AC-11 — Failure-oriented evidence

The repository contains reproducible evidence for the material synchronization and recovery claims, including adverse/failure scenarios rather than happy paths alone.

### AC-12 — Public reproducibility

A fresh checkout following documented instructions can run the system, automated tests and principal demonstration.

### AC-13 — Honest limitations

Public documentation states the boundaries of what has and has not been demonstrated.

### AC-14 — Portfolio-ready artefact

The finished repository contains enough Product, architecture, implementation and validation material for a technically competent reviewer to understand the work without reconstructing it from private conversation.

---

## 15. Product decision boundary

The client/Human Product authority defines:

- Product intent;
- required behaviour;
- Product constraints;
- Product semantics and invariants;
- Product scope;
- Product acceptance outcomes.

The engineering process owns derivable engineering decisions, including:

- architecture;
- implementation stack;
- technical decomposition;
- implementation techniques;
- merge/concurrency mechanisms;
- persistence technologies;
- synchronization protocol implementation;
- testing and validation strategy;
- bounded engineering repair.

A material unresolved ambiguity that would change the intended Product outcome must return to Product authority.

Ordinary reversible engineering uncertainty does not create a Product decision merely because several technically valid solutions exist.

---

## 16. Natural Product backlog

The operational backlog is maintained separately from this frozen Product Definition.

It must represent work that exists because this Product needs it, independent of any external evaluation purpose.

Backlog ordering and Ready state may evolve under legitimate Product authority without rewriting this frozen definition unless Product Truth itself changes.

---

## Product completion criterion

The commission is complete when there is a working, documented, reproducible, public and portfolio-ready Local-first Collaborative Workboard that:

- remains usefully operable through the supported offline boundary;
- preserves supported local intent;
- synchronizes safely after reconnect;
- handles retries without unintended duplicate domain effects;
- supports useful compatible concurrent collaboration;
- detects and surfaces material incompatible concurrency;
- supports explicit durable conflict resolution;
- converges after synchronization and conflict resolution;
- survives the defined restart and recovery scenarios;
- supports stale-client catch-up and local confirmed-state rebuild;
- makes synchronization and failure state intelligible;
- and demonstrates those claims through automated, failure-oriented and real end-to-end evidence.

Technical sophistication that does not contribute to this criterion is not Product completion.
