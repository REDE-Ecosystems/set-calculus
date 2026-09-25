# Polynomial-Time Reduction: 3SAT to RTP

## Status

Completed NP-hardness reduction for the canonical bounded Resolution Transition Problem under `RTP-ENC-v1`.

The repository establishes:

```text
RTP in NP
3SAT <=p RTP_SAT
RTP_SAT subseteq RTP
RTP is NP-hard
RTP is NP-complete
```

All PO-1 through PO-12 correctness obligations are discharged. Canonical encoding and final size/construction proofs are in `RTP_CANONICAL_ENCODING_AND_SIZE_PROOFS.md`.

## 1. Source problem

Let

```text
phi = C_1 AND C_2 AND ... AND C_m
```

be a 3-CNF formula over variables

```text
x_1, ..., x_n
```

where each clause contains exactly three literals.

Define:

```text
3SAT = {
  <phi> |
  exists assignment alpha such that alpha satisfies phi
}
```

The reduction constructs in polynomial time an RTP instance

```text
f(phi) = I_phi
```

such that:

```text
phi in 3SAT
iff
I_phi in RTP
```

## 2. Reduction strategy

The RTP trajectory is divided into two phases:

```text
assignment phase
      |
      v
variable choices
      |
      v
Klein transition
      |
      v
clause validation
      |
      v
reconciliation
```

The assignment phase chooses one Boolean value for each source variable.

The clause phase requires the trajectory to provide one satisfied literal for each source clause.

The Klein transition separates candidate construction from candidate validation:

```text
assignment -> K -> verification
```

## 3. Constructed RTP instance

Given

```text
phi = C_1 AND ... AND C_m
```

construct:

```text
I_phi =
<G_phi, chi, Sigma_A, R_SAT, emptyset, s, t, K, sigma_0, T_R>
```

No nontrivial closure boundaries are required:

```text
B = emptyset
```

If successful, the reduction therefore establishes hardness even for a closure-free RTP subclass.

## 4. Graph construction

Construct a layered directed graph:

```text
G_phi = <V_phi, E_phi>
```

### 4.1 Assignment region

Create assignment-stage vertices:

```text
a_0, a_1, ..., a_n
```

with:

```text
s = a_0
```

For each variable x_i create two choice vertices:

```text
T_i
F_i
```

and edges:

```text
a_(i-1) -> T_i
a_(i-1) -> F_i
T_i -> a_i
F_i -> a_i
```

Thus every complete path through the assignment region chooses exactly one Boolean branch for every variable.

### 4.2 Klein transition

Create one distinguished vertex:

```text
k
```

Set:

```text
K = {k}
```

Create clause-entry vertex c_0 and add:

```text
a_n -> k
k -> c_0
```

Every complete s-to-t trajectory therefore crosses K.

### 4.3 Clause region

For every clause:

```text
C_j = (literal_j1 OR literal_j2 OR literal_j3)
```

create clause terminal vertex:

```text
c_j
```

and literal vertices:

```text
L_j1
L_j2
L_j3
```

Add:

```text
c_(j-1) -> L_jr
L_jr -> c_j
```

for r in {1,2,3}.

A structural path may choose any literal branch. Admissibility determines whether that literal is true under the assignment established before K.

Finally create terminal t and add:

```text
c_m -> t
```

## 5. Assignment representation

For every variable x_i define mutually exclusive relation tokens:

```text
Assign(x_i, TRUE)
Assign(x_i, FALSE)
```

Traversal through T_i establishes:

```text
Assign(x_i, TRUE)
```

Traversal through F_i establishes:

```text
Assign(x_i, FALSE)
```

The opposite assignment relation may not also become active.

These relations persist through the active relational structure and provenance for the remainder of the trajectory.

They induce the assignment alpha_pi:

```text
alpha_pi(x_i) = TRUE  iff Assign(x_i, TRUE)
alpha_pi(x_i) = FALSE iff Assign(x_i, FALSE)
```

## 6. Fixed RTP rule set

The reduction must not generate a formula-specific hidden SAT solver.

Use a fixed finite rule schema:

```text
R_SAT
```

whose semantics are independent of the size of phi. The source formula contributes only graph structure and labels.

### S1. Choose True

At T_i, ChooseTrue(i) is admissible iff neither assignment relation for x_i has previously been established.

Effect:

```text
Assign(x_i, TRUE)
```

### S2. Choose False

At F_i, ChooseFalse(i) is admissible under the same exclusivity condition.

Effect:

```text
Assign(x_i, FALSE)
```

### S3. Positive Literal Validation

If L_jr represents positive literal x_i, entry into L_jr is admissible iff:

```text
Assign(x_i, TRUE)
```

is active.

### S4. Negative Literal Validation

If L_jr represents negative literal NOT x_i, entry into L_jr is admissible iff:

```text
Assign(x_i, FALSE)
```

is active.

### S5. Clause Reconciliation

Successful traversal:

```text
c_(j-1) -> L_jr -> c_j
```

through an admissible literal causes:

```text
sigma_A(c_j) = RECONCILED
```

The selected literal and supporting assignment relation remain in provenance.

### S6. Klein Passage

Traversal through k records that the required interface was crossed and does not modify the assignment.

### S7. Assignment Persistence

Once established:

```text
Assign(x_i, b)
```

for b in {TRUE,FALSE}, remains recoverable throughout the remainder of the trajectory.

No later transition may replace it with its complement.

## 7. Reconciliation target

Set:

```text
T_R = {c_1, ..., c_m}
```

Terminal reconciliation therefore requires:

```text
for every j:
sigma_A(c_j) = RECONCILED
```

Every source clause must therefore have been traversed through at least one admissible literal branch.

Reaching t alone is insufficient.

## 8. Polynomial construction size

Let:

```text
L = |<phi>|
```

The construction contains:

- n+1 assignment-stage vertices;
- 2n variable-choice vertices;
- 1 Klein vertex;
- m+1 clause-stage vertices;
- 3m literal vertices;
- 1 terminal vertex.

Therefore:

```text
|V_phi| = O(n+m)
|E_phi| = O(n+m)
|I_phi| = poly(L)
```

The construction can be emitted in one traversal of the source formula.

Hence:

```text
f(phi)
```

is computable in polynomial time.

## 9. Linear RTP trajectory bound

Every successful trajectory chooses one assignment branch for every variable and one literal branch for every clause.

Under the graph above, a successful trajectory has:

```text
2n + 2m + 3
```

transitions.

The explicit instance encoding contains at least enough vertex, edge, label, and rule information that:

```text
N = |<I_phi>| >= 2n + 2m + 3
```

under the intended canonical encoding.

Therefore:

```text
trajectory length <= N
```

and the reduction fits the canonical RTP linear trajectory bound.

This inequality remains a formal encoding obligation until the canonical finite encoding is frozen.

## 10. Completeness direction

Required theorem direction:

```text
phi in 3SAT
=>
I_phi in RTP
```

Assume alpha satisfies phi.

For each variable x_i:

- traverse T_i if alpha(x_i)=TRUE;
- traverse F_i if alpha(x_i)=FALSE.

This establishes exactly the assignment relations corresponding to alpha.

Traverse:

```text
a_n -> k -> c_0
```

Since alpha satisfies phi, every clause C_j contains at least one literal that is true under alpha.

Choose one such literal vertex L_jr in each clause gadget.

S3 or S4 therefore admits that branch.

S5 reconciles c_j.

After all clauses:

```text
for every c_j in T_R:
sigma_A(c_j) = RECONCILED
```

The trajectory reaches t, crosses K, respects the rules, preserves provenance, and fits the linear bound.

Therefore:

```text
I_phi in RTP
```

## 11. Soundness direction

Required theorem direction:

```text
I_phi in RTP
=>
phi in 3SAT
```

Assume I_phi has an accepted RTP certificate.

Because the graph is layered, every s-to-t trajectory traverses every variable gadget in order.

Each variable gadget therefore selects one of T_i or F_i.

S1, S2, and S7 induce one consistent Boolean assignment alpha_pi.

The path then crosses k in K.

For every clause C_j, the trajectory selects one literal node L_jr.

Acceptance requires S3 or S4 to hold, so that selected literal is true under alpha_pi.

Terminal reconciliation requires every c_j in T_R to be RECONCILED.

Therefore every source clause contains a true literal under alpha_pi:

```text
alpha_pi satisfies phi
```

Hence:

```text
phi in 3SAT
```

## 12. Correctness obligations

The following obligations must be discharged before the reduction is promoted from candidate to theorem.

### PO-1. Canonical encoding — PASS

Discharged by:

```text
RTP_CANONICAL_ENCODING_AND_SIZE_PROOFS.md
```

Under `RTP-ENC-v1`, every graph vertex has an explicit nonempty record. Since `|V_phi| = 3n + 4m + 4`, the encoded instance length satisfies `N >= |V_phi| > 2n + 2m + 3`.

### PO-2. Assignment memory — PASS

Discharged by:

```text
RTP_ACTIVE_RELATIONAL_LEDGER.md
```

Boolean assignments are ordinary `Assign(variable,value)` relational facts stored in the canonical append-only Active Relational Ledger and recovered through `AssignmentView(Lambda_i)`.

### PO-3. Fixed rule-set semantics — PASS

Discharged by:

```text
RTP_SAT_RULE_COMPLEXITY.md
```

`R_SAT` contains exactly seven fixed schemas. Formula-specific information appears only as finite graph structure, labels, identifiers, and targets; it does not alter the rule semantics.

### PO-4. Polynomial primitive checks — PASS

Discharged by:

```text
RTP_SAT_RULE_COMPLEXITY.md
```

Every primitive used by S1-S7 is polynomial-time. Under a conservative unindexed representation, each complete `R_SAT` rule check is `O(N)`; indexing may improve this but is not required.

### PO-5. Exclusivity — PASS

Discharged by:

```text
RTP_SAT_ASSIGNMENT_INVARIANTS.md
```

No accepted trajectory can establish both:

```text
Assign(x_i, TRUE)
Assign(x_i, FALSE)
```

for the same variable.

### PO-6. Assignment completeness — PASS

Discharged by:

```text
RTP_SAT_ASSIGNMENT_INVARIANTS.md
```

Every accepted s-to-t trajectory establishes exactly one assignment value for every variable before crossing K.

### PO-7. Clause soundness — PASS

Discharged by:

```text
RTP_SAT_CLAUSE_SOUNDNESS.md
```

For every clause target, `RECONCILED` can occur only through S5 after traversal of a literal that is true under the total assignment established at K.

### PO-8. No bypass path — PASS

Discharged by:

```text
RTP_SAT_CLAUSE_SOUNDNESS.md
```

The layered graph topology forces every s-to-t trajectory through every variable gadget, K, and every clause gadget in order.

### PO-9. Terminal equivalence — PASS

Discharged by:

```text
RTP_SAT_CLAUSE_SOUNDNESS.md
```

For the constructed instance:

```text
Recon(T_R, q_m)
iff
every source clause has a satisfied selected literal
```

with each reconciliation supported by an incorporated ClauseWitness.

### PO-10. Provenance monotonicity — PASS

Discharged by:

```text
RTP_SAT_PROVENANCE_MONOTONICITY.md
```

Every accepted transition appends a canonical verifier-derived provenance record, preserves the prior provenance prefix unchanged, binds each assertion to its actual rule/transition/witness lineage, and rejects forged, future, nonexistent, overwritten, or erased ancestry.

### PO-11. Certificate bounds — PASS

Discharged by:

```text
RTP_CANONICAL_ENCODING_AND_SIZE_PROOFS.md
```

Every successful reduction trajectory has exactly `2n + 2m + 3 <= N` transitions, and the canonical certificate encoding satisfies `|C| <= 8N^2`.

### PO-12. Polynomial construction — PASS

Discharged by:

```text
RTP_CANONICAL_ENCODING_AND_SIZE_PROOFS.md
```

For source length `L`, the reduction emits `N = O(L log(L+2))` bits and runs in `O(L^2)` time under a conservative bit-cost model.

## 13. Restricted hardness target

The construction uses:

- no nontrivial closure boundaries;
- an acyclic layered graph;
- no trajectory revisits;
- Boolean assignment persistence;
- local literal validation.

Define this restricted subclass as:

```text
RTP_SAT subseteq RTP
```

The proof obligations are discharged, so:

```text
3SAT <=p RTP_SAT
```

and therefore `RTP_SAT`, and hence `RTP`, is NP-hard.

The additional expressive machinery of general RTP is not required for the candidate hardness result.

## 14. Theorem

> **The canonical bounded Resolution Transition Problem under RTP-ENC-v1 is NP-hard under polynomial-time many-one reductions.**

Proof basis:

```text
3SAT <=p RTP
```

Together with:

```text
RTP in NP
```

this yields:

```text
RTP is NP-complete
```

## 15. Reduction summary

```text
3SAT formula
     |
     v
variable gadgets
 choose TRUE/FALSE
     |
     v
persistent assignment
     |
     v
Klein transition
     |
     v
clause gadgets
 choose one literal
     |
     v
literal must agree
with assignment
     |
     v
reconcile every clause
     |
     v
terminal t
```

Core correspondence:

```text
satisfying Boolean assignment
<=>
admissible RTP resolution trajectory

all clauses satisfied
<=>
all clause targets RECONCILED
```
