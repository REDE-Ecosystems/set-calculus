# RTP Active Relational Ledger

## Status

Canonical verifier-state formalization for the Resolution Transition Problem.

This document makes explicit the active relational state already implied by RTP relation deltas, relational consistency checking, provenance persistence, and the Anchor configuration's active relational structure.

It introduces no new Anchor primitive.

For the candidate `3SAT <=p RTP` reduction, it gives the exact representation and persistence semantics for Boolean assignments and discharges **PO-2: Assignment Memory**.

## 1. Notation separation

Distinguish:

```text
E_G
```

for trajectory edges in the finite RTP graph:

```text
G = <V,E_G>
```

from:

```text
Rel(I)
```

the finite universe of typed relational facts representable by RTP instance `I`.

A relational fact has finite encoding:

```text
r = <predicate, arg_1, ..., arg_k>
```

## 2. Active Relational Ledger

At verifier step `i`, define:

```text
Lambda_i = <e_1, ..., e_h>
```

where each event is either:

```text
ASSERT(r, mu)
RETRACT(r, mu)
```

with:

- `r in Rel(I)`;
- `mu` provenance metadata naming the transition, rule, witness, and prior state responsible for the event.

The ledger is append-only:

```text
Lambda_i <=prefix Lambda_(i+1)
```

No transition deletes a prior event.

## 3. Active relation view

Define:

```text
Active(Lambda_i)
```

as the current relational state.

For relation `r`, inspect the most recent ledger event concerning `r`.

`r` is active iff that event is `ASSERT(r,mu)`.

If the latest event is `RETRACT(r,mu)`, then `r` is inactive.

Thus:

```text
active relational state
!=
historical provenance
```

Retraction changes current activity without deleting history.

## 4. Relation to Anchor configuration

The Anchor configuration is:

```text
q = <x, sigma_A, A_E, B_A, P_q>
```

For RTP:

```text
A_E(q_i) = Active(Lambda_i)
```

A verifier configuration may therefore be written:

```text
S_i = <sigma_i, Lambda_i, B_i, P_i>
```

and:

```text
q_i = <v_i, sigma_i, Active(Lambda_i), B_i, P_i>
```

The ledger is therefore the finite computational representation of the Anchor's existing active relational structure.

## 5. Relation delta

Refine each existing RTP certificate delta:

```text
delta_i =
<
  delta_i^sigma,
  delta_i^Lambda,
  delta_i^B,
  delta_i^P
>
```

where `delta_i^Lambda` is the sequence of ledger events authorized at step `i`.

Then:

```text
Lambda_(i+1)
=
Lambda_i || delta_i^Lambda
```

where `||` is append.

A certificate may not invent arbitrary ledger effects.

Require:

```text
AuthorizedEffect(rule_i, delta_i^Lambda) = TRUE
```

as part of `RuleOK`.

## 6. Boolean assignment facts

For the 3SAT reduction use the fixed binary predicate:

```text
Assign(variable,value)
```

with:

```text
value in {0,1}
1 = TRUE
0 = FALSE
```

Thus:

```text
Assign(x_j,1)
Assign(x_j,0)
```

are ordinary relational facts in `Rel(I_phi)`.

They are not new RTP state primitives.

## 7. Exact Boolean assignment view

Define:

```text
beta_i : X_var -> {UNASSIGNED, FALSE, TRUE, CONFLICT}
```

by the active ledger view:

```text
beta_i(x_j) = UNASSIGNED
iff neither Assign(x_j,0) nor Assign(x_j,1) is active

beta_i(x_j) = TRUE
iff Assign(x_j,1) is active
and Assign(x_j,0) is not active

beta_i(x_j) = FALSE
iff Assign(x_j,0) is active
and Assign(x_j,1) is not active

beta_i(x_j) = CONFLICT
iff both are active
```

Accepted `RTP_SAT` trajectories require:

```text
for all x_j:
beta_i(x_j) != CONFLICT
```

Initially:

```text
Lambda_0 = <>
beta_0(x_j) = UNASSIGNED
```

for every source variable.

## 8. Assignment transitions

### ChooseTrue(j)

Precondition:

```text
beta_i(x_j) = UNASSIGNED
```

Authorized relation effect:

```text
ASSERT(Assign(x_j,1), mu_i)
```

Postcondition:

```text
beta_(i+1)(x_j) = TRUE
```

### ChooseFalse(j)

Precondition:

```text
beta_i(x_j) = UNASSIGNED
```

Authorized relation effect:

```text
ASSERT(Assign(x_j,0), mu_i)
```

Postcondition:

```text
beta_(i+1)(x_j) = FALSE
```

All other assignment values are unchanged.

## 9. Assignment persistence

For `R_SAT`:

```text
PersistentPredicate(Assign) = TRUE
```

No `R_SAT` rule authorizes:

```text
RETRACT(Assign(x_j,b), mu)
```

for either Boolean value `b`.

Therefore:

```text
beta_i(x_j) = b
=>
for all k >= i:
beta_k(x_j) = b
```

for accepted `RTP_SAT` trajectories.

## 10. Klein preservation

The Klein transition has:

```text
delta_K^Lambda = <>
```

Therefore:

```text
Lambda_afterK = Lambda_beforeK
beta_afterK = beta_beforeK
```

The Boolean assignment selected before the Klein interface is exactly the assignment available during clause validation.

## 11. Literal validation

For positive literal `x_j`:

```text
LiteralTrue(x_j, Lambda_i)
iff beta_i(x_j) = TRUE
```

For negative literal `NOT x_j`:

```text
LiteralTrue(NOT x_j, Lambda_i)
iff beta_i(x_j) = FALSE
```

Literal validation is therefore a local ledger lookup. No SAT solver is hidden inside `RuleOK`.

## 12. Provenance

For:

```text
ASSERT(Assign(x_j,b), mu_i)
```

use:

```text
mu_i = <step_i, vertex_i, rule_i, witness_i>
```

The provenance state retains the event or a reconstructible reference to it.

Thus:

```text
Lambda_i <=prefix Lambda_(i+1)
P_i <=_P P_(i+1)
```

Assignment persistence and provenance persistence remain distinct:

```text
ledger    = what is active
provenance = why it is active and where it came from
```

## 13. Exact RTP state for the 3SAT reduction

The verifier state is:

```text
S_i = <sigma_i, Lambda_i, B_i, P_i>
```

For the closure-free 3SAT construction:

```text
B_i = emptyset
```

so:

```text
S_i^SAT = <sigma_i, Lambda_i, P_i>
```

The Boolean assignment is not stored twice.

It is the deterministic view:

```text
beta_i = AssignmentView(Lambda_i)
```

## 14. State at the Klein interface

After all `n` variable gadgets on an accepted assignment-phase trajectory:

```text
for all j in {1,...,n}:
beta_K(x_j) in {FALSE,TRUE}
```

and exactly one active assignment fact exists per variable.

This induces:

```text
alpha_pi(x_j) = beta_K(x_j)
```

which remains invariant throughout the clause-validation region.

## 15. Complexity

Each assignment relation stores:

```text
<Assign, variable-index, Boolean-value>
```

requiring:

```text
O(log n)
```

bits.

At most `n` assignment assertions occur, so the assignment ledger requires:

```text
O(n log n)
```

bits.

Since `n <= N`, this is polynomial and within the canonical `O(N^2)` certificate bound.

Indexed lookup is:

```text
O(1) expected
```

with hashing or:

```text
O(log n)
```

deterministically with an ordered map.

## 16. PO-2 discharge

PO-2 requires that `Assign(x_i,TRUE/FALSE)` be representable using canonical RTP active relation/state/provenance machinery without an unmodeled primitive.

The representation is:

```text
authoritative storage:
  Lambda_i

Boolean relational facts:
  Assign(x_i,0)
  Assign(x_i,1)

derived state:
  beta_i = AssignmentView(Lambda_i)

persistence:
  no R_SAT rule authorizes RETRACT(Assign(...))

provenance:
  every ASSERT event retains step + rule + witness lineage
```

Therefore:

```text
PO-2 Assignment Memory = PASS
```

No additional computational primitive is required.
