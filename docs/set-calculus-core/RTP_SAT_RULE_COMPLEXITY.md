# RTP SAT Fixed Rule Schema and Polynomial Primitive Verification

## Status

Formal proof artifact for the candidate `3SAT <=p RTP` reduction.

This document proves:

```text
PO-3 Fixed Rule-Set Semantics      = PASS
PO-4 Polynomial Primitive Checks   = PASS
```

for the restricted reduction subclass `RTP_SAT`.

It depends on:

- `RTP_NP_HARDNESS_REDUCTION.md`;
- `RTP_ACTIVE_RELATIONAL_LEDGER.md`;
- `RTP_SAT_ASSIGNMENT_INVARIANTS.md`;
- `RTP_SAT_CLAUSE_SOUNDNESS.md`.

The proof separates **fixed rule semantics** from **formula-specific encoded data**.

---

# 1. Goal

For a source formula:

```text
phi = C_1 AND ... AND C_m
```

the reduction constructs:

```text
I_phi =
<G_phi, chi, Sigma_A, R_SAT, emptyset, s, t, K, sigma_0, T_R>
```

PO-3 requires that:

```text
R_SAT
```

be a fixed finite rule schema independent of `phi`.

PO-4 requires every primitive rule check used by `R_SAT` to be decidable in polynomial time in:

```text
N = |<I_phi>|.
```

---

# 2. Formula-specific data versus fixed semantics

The reduction may encode formula-specific **data**.

It may not generate formula-specific **algorithms**.

For `I_phi`, the following may vary with `phi`:

```text
G_phi                  graph topology
vertex identifiers     a_i, T_i, F_i, L_jr, c_j
literal labels         variable index + polarity
T_R                    clause-target identifiers
n                       number of variables
m                       number of clauses
```

The following do not vary with `phi`:

```text
the number of rule schemas
the meaning of each rule schema
the precondition form of each rule schema
the authorized effect form of each rule schema
the Boolean domain {0,1}
the predicates Assign and ClauseWitness
the interpretation of RECONCILED
```

Thus:

```text
formula data
!=
rule semantics.
```

---

# 3. Canonical vertex labels

The graph carries finite typed labels.

Use the fixed label alphabet:

```text
START
ASSIGN_STAGE(i)
TRUE_CHOICE(i)
FALSE_CHOICE(i)
KLEIN
CLAUSE_STAGE(j)
POS_LITERAL(j,r,h)
NEG_LITERAL(j,r,h)
TERMINAL
```

where:

- `i` is a variable index;
- `j` is a clause index;
- `r in {1,2,3}` is a literal position;
- `h` identifies the source variable referenced by that literal.

These labels are data records.

For example:

```text
POS_LITERAL(7,2,4)
```

means:

```text
the second literal of clause 7 is x_4.
```

It does not encode an algorithm for deciding whether the clause is satisfied.

---

# 4. Fixed rule-schema set

Define:

```text
R_SAT = {
  S1 ChooseTrue,
  S2 ChooseFalse,
  S3 ValidatePositiveLiteral,
  S4 ValidateNegativeLiteral,
  S5 ClauseReconcile,
  S6 KleinPass,
  S7 AssignmentPersistence
}
```

Therefore:

```text
|R_SAT| = 7.
```

This cardinality is independent of:

```text
n,
m,
|phi|,
N.
```

The schemas may be instantiated with indices read from graph labels.

An instantiation such as:

```text
ChooseTrue(i=17)
```

is an application of S1.

It is not a seventeenth or formula-specific rule.

---

# 5. S1 ChooseTrue

## Schema

Input label:

```text
TRUE_CHOICE(i)
```

Precondition:

```text
beta(x_i) = UNASSIGNED
```

Authorized relational effect:

```text
ASSERT(Assign(x_i,1))
```

No other assignment effect is permitted.

## Formula independence

The semantics are identical for every variable index.

The schema does not inspect any clause and does not inspect the source formula.

Only the parameter `i` varies.

---

# 6. S2 ChooseFalse

## Schema

Input label:

```text
FALSE_CHOICE(i)
```

Precondition:

```text
beta(x_i) = UNASSIGNED
```

Authorized relational effect:

```text
ASSERT(Assign(x_i,0))
```

The semantics are identical for all `i`.

---

# 7. S3 ValidatePositiveLiteral

## Schema

Input label:

```text
POS_LITERAL(j,r,h)
```

Precondition:

```text
beta(x_h) = TRUE
```

Authorized assignment-ledger effect:

```text
none
```

The rule asks only whether one previously established relational fact is active:

```text
Assign(x_h,1).
```

It does not ask whether:

```text
C_j
```

or:

```text
phi
```

is satisfiable.

---

# 8. S4 ValidateNegativeLiteral

## Schema

Input label:

```text
NEG_LITERAL(j,r,h)
```

Precondition:

```text
beta(x_h) = FALSE
```

Authorized assignment-ledger effect:

```text
none
```

Again, only one stored assignment value is tested.

---

# 9. S5 ClauseReconcile

## Schema

Parameters are read from the local graph labels:

```text
ClauseReconcile(j,r,h,b)
```

Preconditions:

```text
current local path =
c_(j-1) -> L_jr -> c_j

LiteralAdmissible(L_jr,Lambda) = TRUE
```

Authorized state effect:

```text
sigma_A(c_j) := RECONCILED
```

Authorized relational effect:

```text
ASSERT(
  ClauseWitness(
    c_j,
    L_jr,
    Assign(x_h,b)
  )
)
```

where `b` is the Boolean value required by the literal polarity.

No other `R_SAT` rule may set a clause target to `RECONCILED`.

S5 performs only local validation of the branch already selected by the trajectory.

It does not search the three literals of a clause for a satisfying one.

That selection is represented by the trajectory itself.

---

# 10. S6 KleinPass

## Schema

Preconditions:

```text
current vertex label = KLEIN
current vertex in K
```

Authorized assignment-ledger effect:

```text
empty
```

The rule records the interface crossing and preserves the assignment ledger unchanged.

Its semantics do not depend on `phi`.

---

# 11. S7 AssignmentPersistence

S7 is a fixed admissibility invariant over transition deltas.

For every proposed transition delta:

```text
delta_i^Lambda
```

require:

```text
no event in delta_i^Lambda has form:
RETRACT(Assign(x,b))
```

and require that no rule other than S1 or S2 asserts an `Assign` fact.

This invariant has the same semantics for every RTP_SAT instance.

It does not enumerate variables or clauses in the rule definition.

---

# 12. Theorem PO-3: Fixed Rule-Set Semantics

**Theorem.**

`R_SAT` is a fixed finite rule schema independent of the source 3-CNF formula.

### Proof

The rule family contains exactly seven schemas:

```text
S1 through S7.
```

Their syntax and semantics are fixed above.

For every source formula `phi`, the reduction varies only finite graph data:

- the number of variable gadgets;
- the number of clause gadgets;
- vertex identifiers;
- literal labels;
- graph edges;
- target identifiers.

Rule applications receive indices such as `i`, `j`, `r`, and `h` from those labels.

Parameterized application does not increase the number of rule schemas.

No schema contains:

- the text of `phi`;
- a list of all clauses;
- an enumeration of candidate assignments;
- recursion over assignments;
- a call to a SAT oracle;
- a search for a satisfying literal.

Literal choice is made by the candidate trajectory.

Each rule checks only the local choice already presented to the verifier.

Therefore:

```text
|R_SAT| = 7 = O(1)
```

for every reduced instance.

Thus:

```text
PO-3 Fixed Rule-Set Semantics = PASS.
```

QED.

---

# 13. Primitive operations used by RuleOK

For RTP_SAT, every rule check is composed from the following primitive operations:

```text
P1  DecodeLabel(v)
P2  EdgeMember(u,v)
P3  AssignmentView(x,Lambda)
P4  AuthorizedRelationEffect(rule,delta^Lambda)
P5  AuthorizedStateEffect(rule,delta^sigma)
P6  LiteralPolarityAndVariable(L)
P7  AppendLedgerEvent
P8  AppendProvenanceReference
P9  StateRead(c_j)
P10 StateWriteAuthorized(c_j,RECONCILED)
P11 IsKlein(v)
P12 ScanDeltaForForbiddenAssignmentRetraction
```

No primitive invokes a global search procedure.

---

# 14. Size bounds available to primitive checks

Let:

```text
N = |<I_phi>|.
```

For the reduced instance:

```text
|V_phi| <= N
|E_phi| <= N
n <= N
m <= N
```

up to ordinary encoding constants.

An identifier or index uses at most:

```text
O(log N)
```

bits.

Each accepted RTP_SAT transition produces only a constant number of state, relational, and provenance effects.

The trajectory has at most `N` transitions.

Therefore the active ledger/provenance material accumulated during the restricted reduction contains at most:

```text
O(N)
```

constant-arity events.

Even without indexing, a complete scan of the current RTP_SAT ledger is therefore:

```text
O(N)
```

event comparisons, with polynomial bit cost.

This is sufficient for PO-4.

Indexes improve the bound but are not required for polynomiality.

---

# 15. Complexity of primitive operations

## P1 DecodeLabel

A label is part of the encoded instance and has length at most `N`.

Therefore:

```text
DecodeLabel = O(N)
```

under a maximally conservative bit-scan bound.

With the canonical compact labels used by the reduction it is:

```text
O(log N).
```

## P2 EdgeMember

Without an index, scanning the finite edge encoding requires at most:

```text
O(N)
```

encoded records.

Thus:

```text
EdgeMember = poly(N).
```

With adjacency indexing it is `O(log N)` deterministic or `O(1)` expected.

## P3 AssignmentView

The query needs to determine whether:

```text
Assign(x,0)
Assign(x,1)
```

are active.

A scan of the `O(N)` RTP_SAT ledger is:

```text
O(N)
```

record checks.

With an ordered active-relation map:

```text
O(log N).
```

## P4 AuthorizedRelationEffect

Each R_SAT rule authorizes a constant-size effect pattern.

Comparing the proposed delta to that pattern requires scanning at most the certificate-supplied transition delta.

The canonical RTP bound gives:

```text
|delta_i| <= N.
```

Therefore:

```text
AuthorizedRelationEffect = O(N)
```

in the general representation and constant-record work for well-formed RTP_SAT deltas.

## P5 AuthorizedStateEffect

The authorized state-effect set for a rule schema is constant-size.

A proposed state delta has length at most `N`.

Therefore:

```text
AuthorizedStateEffect = O(N).
```

## P6 LiteralPolarityAndVariable

The literal node label directly stores:

```text
polarity
variable index
```

so decoding is at most:

```text
O(N)
```

and `O(log N)` for the compact reduction encoding.

## P7 AppendLedgerEvent

Appending a constant-size event to an append-only representation is polynomial.

Even under immutable copying:

```text
O(N)
```

is sufficient for the restricted ledger size.

## P8 AppendProvenanceReference

Each RTP_SAT transition appends only a constant number of bounded references.

Hence the operation is polynomial, conservatively:

```text
O(N).
```

## P9 StateRead

There are at most `O(N)` encoded state entries.

A linear scan gives:

```text
O(N).
```

## P10 StateWriteAuthorized

Authorization is a constant-schema check plus a bounded state update.

Thus:

```text
O(N)
```

conservatively.

## P11 IsKlein

Testing whether the current node is the unique constructed Klein node can be performed by comparing its identifier or label.

Thus:

```text
O(N)
```

conservatively and `O(log N)` under compact identifiers.

## P12 ScanDeltaForForbiddenAssignmentRetraction

The transition delta has encoded length at most `N`.

Scanning it for:

```text
RETRACT(Assign(...))
```

therefore costs:

```text
O(N).
```

---

# 16. Rule-by-rule complexity

Using the primitive bounds above:

| Rule | Required checks | Conservative bound |
|---|---|---:|
| S1 ChooseTrue | label, edge, AssignmentView, authorized effect, S7 | `O(N)` |
| S2 ChooseFalse | label, edge, AssignmentView, authorized effect, S7 | `O(N)` |
| S3 ValidatePositiveLiteral | label, edge, polarity/variable, AssignmentView | `O(N)` |
| S4 ValidateNegativeLiteral | label, edge, polarity/variable, AssignmentView | `O(N)` |
| S5 ClauseReconcile | local edges, literal admissibility, state/effect authorization, witness provenance | `O(N)` |
| S6 KleinPass | label/K membership, edge, empty assignment effect | `O(N)` |
| S7 AssignmentPersistence | scan transition delta for forbidden assignment mutation | `O(N)` |

Hence every fixed R_SAT schema application satisfies:

```text
RuleCheck_Sk(N) = O(N)
```

under conservative unindexed representations.

A more efficient implementation may achieve:

```text
O(log N)
```

or expected:

```text
O(1)
```

for many ledger and graph lookups, but those stronger implementation bounds are unnecessary to prove PO-4.

---

# 17. Composite RuleOK bound

For a proposed RTP_SAT transition, `RuleOK`:

1. decodes the local labels;
2. identifies one of the seven rule schemas;
3. checks graph adjacency;
4. evaluates that schema's local preconditions;
5. validates the proposed state/ledger effects;
6. validates the fixed persistence invariant;
7. validates bounded provenance references.

The number of primitive checks per transition is constant.

Each primitive check is:

```text
O(N)
```

under the conservative representation.

Therefore:

```text
RuleOK_RSAT(N) = O(N).
```

Even if each low-level bit comparison is expanded conservatively, the bound remains:

```text
N^O(1).
```

Thus no primitive rule check is superpolynomial.

---

# 18. No hidden SAT computation

The verifier never performs:

```text
for each assignment alpha:
  test phi under alpha
```

and never performs:

```text
search literals of C_j until one satisfies C_j.
```

Instead the certificate trajectory supplies:

```text
one assignment branch per variable
one literal branch per clause.
```

The verifier asks only:

```text
is this chosen variable transition legal?
is this chosen literal true under the stored assignment?
is this proposed reconciliation effect authorized?
```

Those are local deterministic checks.

Therefore the NP search resides in the existential trajectory:

```text
exists C
```

rather than inside the deterministic verifier.

---

# 19. Theorem PO-4: Polynomial Primitive Checks

**Theorem.**

Every primitive rule check used by `R_SAT` is decidable in polynomial time in the encoded RTP instance size:

```text
N = |<I_phi>|.
```

### Proof

Sections 13-16 enumerate every primitive operation needed by the seven fixed schemas.

Each operation is bounded by:

```text
O(N)
```

under conservative unindexed finite representations.

Every R_SAT rule invokes only a constant number of those operations.

Therefore for every schema `S_k`:

```text
T_(S_k)(N) = O(N)
```

and hence:

```text
T_(S_k)(N) in N^O(1).
```

Thus all primitive rule checks satisfy RTP acceptance condition R1.

Therefore:

```text
PO-4 Polynomial Primitive Checks = PASS.
```

QED.

---

# 20. Combined conclusion

We have established:

```text
R_SAT has exactly 7 fixed schemas
```

and:

```text
every R_SAT primitive rule check is polynomial-time.
```

Therefore:

```text
PO-3 = PASS
PO-4 = PASS
```

The reduction does not encode an NP-hard solver inside `R_SAT`.

Its combinatorial choice is carried by the candidate RTP trajectory and checked locally by the deterministic verifier.

---

# 21. Proof-obligation status

```text
PO-1  Canonical Encoding            OPEN
PO-2  Assignment Memory             PASS
PO-3  Fixed Rule-Set Semantics      PASS
PO-4  Polynomial Primitive Checks   PASS
PO-5  Assignment Exclusivity        PASS
PO-6  Assignment Completeness       PASS
PO-7  Clause Soundness              PASS
PO-8  No Bypass Path                PASS
PO-9  Terminal Equivalence          PASS
PO-10 Provenance Monotonicity       OPEN
PO-11 Certificate Bounds            OPEN
PO-12 Polynomial Construction       OPEN
```

The remaining open obligations are now limited to finite encoding, provenance monotonicity, certificate-size bounds, and polynomial construction size/time.
