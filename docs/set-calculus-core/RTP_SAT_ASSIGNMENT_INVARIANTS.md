# RTP SAT Assignment Invariants

## Status

Formal proof artifact for the candidate `3SAT <=p RTP` reduction.

This document uses the canonical Active Relational Ledger in:

```text
RTP_ACTIVE_RELATIONAL_LEDGER.md
```

and discharges:

```text
PO-5 Assignment Exclusivity = PASS
PO-6 Assignment Completeness = PASS
```

for the restricted reduction subclass `RTP_SAT`.

## 1. Definitions used

For every source variable `x_i`, the variable gadget is:

```text
             T_i
            /   \
a_(i-1) ---       --- a_i
            \   /
             F_i
```

with directed edges:

```text
a_(i-1) -> T_i
a_(i-1) -> F_i
T_i -> a_i
F_i -> a_i
```

The assignment view is:

```text
beta_j(x_i) in {
  UNASSIGNED,
  FALSE,
  TRUE,
  CONFLICT
}
```

derived entirely from the active relational ledger `Lambda_j`.

The only rules authorized to introduce assignment facts are:

```text
ChooseTrue(i)
ChooseFalse(i)
```

with precondition:

```text
beta_j(x_i) = UNASSIGNED
```

and effects:

```text
ChooseTrue(i)
  -> ASSERT(Assign(x_i,1))

ChooseFalse(i)
  -> ASSERT(Assign(x_i,0))
```

No `R_SAT` rule authorizes assignment retraction.

## 2. Lemma 1: Single-effect assignment rule

**Lemma.** A valid assignment transition for variable `x_i` introduces exactly one of:

```text
Assign(x_i,1)
Assign(x_i,0)
```

and cannot introduce both.

### Proof

The authorized relational effect of `ChooseTrue(i)` is exactly:

```text
ASSERT(Assign(x_i,1))
```

The authorized relational effect of `ChooseFalse(i)` is exactly:

```text
ASSERT(Assign(x_i,0))
```

`RuleOK` requires:

```text
AuthorizedEffect(rule_i, delta_i^Lambda) = TRUE
```

Therefore a certificate delta containing both assignment assertions is not an authorized effect of either assignment rule and is rejected.

Hence every valid assignment transition contributes exactly one Boolean assignment fact.

QED.

## 3. Lemma 2: Assignment irreversibility in RTP_SAT

**Lemma.** Once `beta_j(x_i)` becomes TRUE or FALSE, it retains that value for the remainder of any accepted `RTP_SAT` trajectory.

### Proof

By the Active Relational Ledger specification:

```text
PersistentPredicate(Assign) = TRUE
```

for `R_SAT`.

No rule authorizes:

```text
RETRACT(Assign(x_i,b))
```

for `b in {0,1}`.

No rule other than `ChooseTrue(i)` or `ChooseFalse(i)` may introduce an `Assign` fact.

After either assignment rule executes, the assignment view is no longer:

```text
UNASSIGNED
```

so the precondition of both assignment rules is false for that variable.

Therefore no later valid transition can remove or replace the selected assignment.

QED.

## 4. Theorem PO-5: Assignment Exclusivity

**Theorem.** No accepted `RTP_SAT` trajectory can establish both:

```text
Assign(x_i,1)
```

and:

```text
Assign(x_i,0)
```

for the same variable `x_i`.

Equivalently:

```text
for every accepted step j and variable x_i:
beta_j(x_i) != CONFLICT
```

### Proof

Initially:

```text
beta_0(x_i) = UNASSIGNED
```

Suppose the first assignment transition for `x_i` is `ChooseTrue(i)`.

By Lemma 1 it adds only:

```text
Assign(x_i,1)
```

so afterward:

```text
beta(x_i) = TRUE
```

The precondition:

```text
beta(x_i) = UNASSIGNED
```

for `ChooseFalse(i)` is now false.

By Lemma 2 the TRUE assignment cannot be retracted or replaced.

Therefore `Assign(x_i,0)` cannot subsequently become active.

The argument is symmetric if the first assignment transition is `ChooseFalse(i)`.

A certificate also cannot directly inject the opposite assertion because `AuthorizedEffect` rejects ledger effects not authorized by the selected rule.

Thus both assignment facts can never be simultaneously active in an accepted trajectory.

Therefore:

```text
PO-5 Assignment Exclusivity = PASS
```

QED.

## 5. Lemma 3: Variable-gadget cut property

**Lemma.** Every directed path from `a_(i-1)` to `a_i` in the constructed graph contains exactly one of `T_i` or `F_i`.

### Proof

By construction the only outgoing edges from `a_(i-1)` within the variable layer are:

```text
a_(i-1) -> T_i
a_(i-1) -> F_i
```

The only outgoing edge from `T_i` is:

```text
T_i -> a_i
```

and the only outgoing edge from `F_i` is:

```text
F_i -> a_i
```

There is no edge:

```text
T_i -> F_i
F_i -> T_i
```

and no bypass edge:

```text
a_(i-1) -> a_i
```

Therefore any directed path crossing the variable layer must choose one branch and cannot traverse both branches.

QED.

## 6. Lemma 4: Ordered traversal of all variable gadgets

**Lemma.** Every directed `s`-to-`K` path in the constructed instance traverses the variable gadgets in the order:

```text
x_1, x_2, ..., x_n
```

and traverses every one of them.

### Proof

The assignment-region stage vertices are:

```text
a_0, a_1, ..., a_n
```

with:

```text
s = a_0
```

For each `i`, the only way to advance from stage `a_(i-1)` to the next stage `a_i` is through the corresponding `T_i/F_i` gadget by Lemma 3.

There are no edges from an earlier stage directly to a later stage.

The only edge from the final assignment stage toward the Klein region is:

```text
a_n -> k
```

Therefore any path reaching `k in K` from `s` must pass through every variable gadget exactly once and in increasing index order.

QED.

## 7. Lemma 5: At-least-one assignment per variable

**Lemma.** Every accepted `s`-to-`K` trajectory establishes at least one Boolean assignment fact for every source variable.

### Proof

By Lemma 4, every accepted trajectory reaching `K` enters exactly one of:

```text
T_i
F_i
```

for each source variable `x_i`.

Entry through `T_i` requires a valid `ChooseTrue(i)` transition, whose authorized effect asserts:

```text
Assign(x_i,1)
```

Entry through `F_i` requires a valid `ChooseFalse(i)` transition, whose authorized effect asserts:

```text
Assign(x_i,0)
```

An accepted RTP certificate cannot traverse the node without satisfying `RuleOK`.

Therefore every source variable acquires at least one Boolean assignment fact before the Klein transition.

QED.

## 8. Theorem PO-6: Assignment Completeness

**Theorem.** Every accepted `s`-to-`t` trajectory in the constructed `RTP_SAT` instance establishes exactly one Boolean assignment for every source variable before crossing the Klein transition.

Formally, if `j_K` is the trajectory index at the Klein node:

```text
for all x_i:
beta_(j_K)(x_i) in {FALSE, TRUE}
```

and:

```text
|Active(Lambda_(j_K)) intersect AssignFacts| = n
```

### Proof

Because every accepted RTP trajectory must satisfy the Klein-crossing condition, it contains `k in K`.

By Lemma 4, the prefix from `s` to `k` traverses every variable gadget.

By Lemma 5, that traversal establishes at least one assignment fact for every variable.

By PO-5 Assignment Exclusivity, no variable can have both Boolean assignment facts active.

Therefore every variable has exactly one assignment value before the trajectory reaches `K`.

Since there are `n` source variables, exactly `n` active assignment facts exist at the Klein interface.

Thus the ledger induces a total Boolean assignment:

```text
alpha_pi(x_i) = beta_(j_K)(x_i)
```

for all `i in {1,...,n}`.

Therefore:

```text
PO-6 Assignment Completeness = PASS
```

QED.

## 9. Corollary: Unique assignment induced by an accepted trajectory

Every accepted `RTP_SAT` trajectory induces exactly one total Boolean assignment:

```text
alpha_pi : {x_1,...,x_n} -> {FALSE,TRUE}
```

at the Klein interface.

Because assignment facts are persistent:

```text
alpha_pi
```

remains unchanged throughout the clause-validation region.

This is the assignment used by all later literal-admissibility checks.

## 10. Proof-obligation status

The reduction proof ledger is now:

```text
PO-1 Canonical Encoding        OPEN
PO-2 Assignment Memory         PASS
PO-3 Fixed Rule-Set Semantics  OPEN
PO-4 Polynomial Primitive Checks OPEN
PO-5 Assignment Exclusivity    PASS
PO-6 Assignment Completeness   PASS
PO-7 Clause Soundness          OPEN
PO-8 No Bypass Path            OPEN
PO-9 Terminal Equivalence      OPEN
PO-10 Provenance Monotonicity  OPEN
PO-11 Certificate Bounds       OPEN
PO-12 Polynomial Construction  OPEN
```

## 11. Dependencies established

The following invariant is now available to the remaining proof:

```text
accepted trajectory
  ->
crosses every variable gadget
  ->
exactly one assignment per variable
  ->
total assignment at K
  ->
same assignment persists after K
```

This directly supplies the assignment side of the later clause-soundness and terminal-equivalence proofs.
