# ENGINEERING_CANDIDATES__NOT_SELECTED

**Status:** CANDIDATES ONLY — NOT SELECTED  
**Authority:** NONE for Product Truth or architecture selection  
**Purpose:** Preserve potentially useful engineering options without prematurely choosing them.

The entries below are not Product requirements, architecture decisions, implementation instructions or backlog commitments.

Their presence records only that they may be considered during legitimate engineering execution.

Selection, rejection or replacement belongs to the Anvil-governed engineering process under the applicable `FIT_FOR_PURPOSE_IMPLEMENTATION_STACK_SELECTION` boundary and the frozen Product contracts.

## Candidate 1 — Deletion outside the initial synchronizable scope

Technical Contract §12 permits either:

- synchronized deletion with durable deletion/tombstone or equivalent semantics; or
- deletion outside initial synchronizable scope.

The second option may reduce initial synchronization complexity.

**Disposition:** `NOT_SELECTED`.

Engineering must decide from Product usefulness, implementation consequences and evidence. This file does not choose Option B.

## Candidate 2 — Field-aware compatibility / conflict rules

A possible implementation may classify concurrent changes using the affected user-visible properties so that some independent changes compose while incompatible changes conflict.

**Disposition:** `NOT_SELECTED`.

The Technical Product Contract requires useful compatible concurrency and material conflict detection, but it does not select field-level merge as the architecture.

## Candidate 3 — Single application backend

A single application backend may be sufficient for the Product and could keep operational complexity low.

**Disposition:** `NOT_SELECTED`.

The Technical Product Contract requires logical server/API responsibilities but does not select a monolith, service layout or framework.

## Candidate 4 — One durable transactional server persistence layer

A single durable store with an appropriate atomicity boundary may be a simple way to satisfy confirmed-write, idempotency and transaction invariants.

**Disposition:** `NOT_SELECTED`.

No database or persistence technology is selected.

## Candidate 5 — Durable client pending-operation queue / outbox

A browser-local durable queue of pending operations may be a useful implementation technique for offline intent and retry.

**Disposition:** `NOT_SELECTED`.

The Product requires durable pending intent, not an outbox pattern by name.

## Reconsideration and selection

Engineering may:

- select one of these candidates;
- reject it;
- narrow it;
- replace it with another technically sufficient approach;
- combine compatible candidates.

The decision must be made for the Workboard and supported proportionately by actual requirements and evidence.

Available Anvil capability, novelty, study value or desire to exercise a technique is not sufficient justification.
