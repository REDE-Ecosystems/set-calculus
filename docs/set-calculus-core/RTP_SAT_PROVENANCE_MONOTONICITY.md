# RTP SAT Provenance Monotonicity

## Status

Formal proof artifact for the candidate `3SAT <=p RTP` reduction.

This document proves:

```text
PO-10 Provenance Monotonicity = PASS
```

for the restricted reduction subclass `RTP_SAT`.

It depends on:

- `RTP_ACTIVE_RELATIONAL_LEDGER.md`;
- `RTP_SAT_RULE_COMPLEXITY.md`;
- the canonical RTP requirement `P_i <=_P P_(i+1)`.

The proof establishes three properties:

1. every accepted transition preserves all prior provenance;
2. every assertion retains its introducing rule, transition, and witness lineage;
3. no later transition can forge, overwrite, or erase that lineage.

---

# 1. Provenance state

For RTP_SAT, represent the provenance state at verifier step `i` as an append-only finite sequence:

```text
P_i = <p_0, p_1, ..., p_(i-1)>
```

where each `p_h` is the canonical provenance record for accepted transition `h`.

Initially:

```text
P_0 = <>
```

For every accepted transition `i`:

```text
P_(i+1) = P_i || <p_i>
```

where `||` denotes append.

Therefore:

```text
P_i <=prefix P_(i+1)
```

for every accepted transition.

The prefix relation is a concrete realization of the Anchor provenance order:

```text
P_i <=_P P_(i+1).
```

No accepted transition replaces the provenance state with an unrelated structure.

---

# 2. Canonical provenance record

For accepted transition:

```text
tau_i =
<
  step_i,
  v_i,
  v_(i+1),
  rule_i,
  params_i,
  delta_i,
  omega_i
>
```

define the verifier-derived provenance record:

```text
p_i =
<
  pid_i,
  step_i,
  from_i,
  to_i,
  rule_i,
  params_i,
  witness_refs_i,
  parent_refs_i,
  effect_refs_i
>
```

with:

```text
from_i = v_i
to_i   = v_(i+1)
```

and:

- `rule_i` is the fixed R_SAT schema selected by RuleOK;
- `params_i` are the parameters decoded from the local graph labels;
- `witness_refs_i` identify validated witness objects used at this transition;
- `parent_refs_i` identify prior provenance records on which the transition depends;
- `effect_refs_i` identify the accepted state and relational effects produced by this transition.

Use the transition index as the canonical provenance identifier:

```text
pid_i = i.
```

Because an RTP certificate has one ordered transition at each index, `pid_i` is unique within the certificate.

No cryptographic assumption is required for this proof.

---

# 3. Verifier-derived provenance

The certificate may supply:

```text
omega_i
```

and proposed transition effects.

It does not get to author an arbitrary accepted provenance record.

After RuleOK validates the transition, the verifier constructs:

```text
p_i = CanonicalProv(I, S_i, v_i, v_(i+1), rule_i, delta_i, omega_i).
```

Equivalently, if a serialized `delta_i^P` is present in the certificate, acceptance requires:

```text
delta_i^P = <CanonicalProv(...)>.
```

Any mismatch is rejected.

Thus provenance is a deterministic consequence of the accepted transition rather than an unaudited claim supplied by the certificate.

---

# 4. Witness validity

Every reference in:

```text
witness_refs_i
```

must point to a witness actually validated by the rule application.

For assignment rules S1 and S2, the witness identifies:

```text
the selected assignment branch
the variable index
the Boolean value
the graph labels authorizing the choice
```

For literal rules S3 and S4, the witness identifies:

```text
the selected literal node
the persistent Assign fact used by LiteralAdmissible
the provenance record that introduced that Assign fact
```

For S5 ClauseReconcile, the witness identifies:

```text
the clause target c_j
the selected literal L_jr
the supporting Assign(x_h,b) fact
the provenance record that introduced Assign(x_h,b)
```

For S6 KleinPass, the witness identifies the verified Klein crossing.

S7 contributes the persistence check applied to the proposed delta.

A witness reference to nonexistent, future, or unvalidated material is rejected.

---

# 5. Parent-reference rule

Every:

```text
parent_ref in parent_refs_i
```

must satisfy:

```text
0 <= parent_ref < i.
```

and must identify an existing provenance record in:

```text
P_i.
```

Therefore provenance dependencies form a directed acyclic graph embedded in the append-only sequence.

No accepted record may claim ancestry from:

- a future transition;
- a nonexistent transition;
- an unrelated record not used by the current rule check.

---

# 6. Assertion-lineage binding

For every ledger assertion:

```text
ASSERT(r, mu_i)
```

accepted in:

```text
delta_i^Lambda,
```

require:

```text
mu_i =
<
  pid_i,
  step_i,
  rule_i,
  witness_refs_i
>
```

and require:

```text
r in effect_refs_i.
```

Thus each assertion is directly bound to the provenance record of the transition that introduced it.

The binding is not merely descriptive.

RuleOK verifies both:

```text
AuthorizedEffect(rule_i, delta_i^Lambda)
```

and:

```text
EffectLineageMatch(delta_i^Lambda, p_i).
```

Acceptance requires both to hold.

---

# 7. Assignment-assertion lineage

For S1 ChooseTrue or S2 ChooseFalse, an accepted assertion has the form:

```text
ASSERT(Assign(x_j,b), mu_i)
```

with canonical lineage:

```text
mu_i =
<
  pid_i,
  step_i,
  ChooseTrue/ChooseFalse,
  assignment-branch witness
>
```

and:

```text
effect_refs_i contains Assign(x_j,b).
```

Therefore, given any active assignment fact, its introducing transition can be reconstructed:

```text
Assign(x_j,b)
  ->
ledger ASSERT event
  ->
pid_i
  ->
p_i
  ->
step i
  ->
graph transition
  ->
R_SAT assignment rule
  ->
validated assignment witness.
```

This lineage survives every later transition.

---

# 8. ClauseWitness lineage

For S5 ClauseReconcile, the accepted relational effect includes:

```text
ASSERT(
  ClauseWitness(
    c_j,
    L_jr,
    Assign(x_h,b)
  ),
  mu_i
)
```

The S5 provenance record contains:

```text
parent_refs_i
```

including the provenance identifier of the earlier assignment assertion:

```text
Assign(x_h,b).
```

Hence a clause witness reconstructs the complete chain:

```text
variable choice
  ->
assignment assertion
  ->
assignment provenance p_h
  ->
literal validation
  ->
S5 transition
  ->
ClauseWitness assertion
  ->
clause reconciliation.
```

The later clause record does not replace the earlier assignment provenance.

It references it.

---

# 9. Provenance-preservation rule

For every accepted RTP_SAT transition require:

```text
ProvPreserve(P_i, P_(i+1))
iff
P_(i+1) = P_i || <p_i>.
```

Consequently every prior record remains at the same position with the same contents:

```text
for all h < i:
P_(i+1)[h] = P_i[h].
```

This gives the stronger property:

```text
ImmutablePrefix(P_i,P_(i+1)).
```

Therefore any information reconstructible from `P_i` remains reconstructible from `P_(i+1)`.

Thus:

```text
P_i <=_P P_(i+1).
```

---

# 10. Retraction does not erase lineage

The general Active Relational Ledger permits:

```text
RETRACT(r,mu).
```

A retraction changes:

```text
Active(Lambda)
```

but does not delete the earlier ASSERT event.

Likewise, the retraction transition appends its own provenance record.

Therefore even in RTP variants that permit retraction:

```text
ASSERT(r)
...
RETRACT(r)
```

retains both historical events and their separate provenance.

For RTP_SAT specifically, assignment assertions cannot be retracted at all.

Thus assignment lineage has both:

- historical provenance persistence;
- active-state persistence.

---

# 11. Lemma 1: Prior provenance cannot be erased

**Lemma.**

For every accepted transition:

```text
S_i -> S_(i+1),
```

every provenance record present in `P_i` remains present, unchanged, in `P_(i+1)`.

### Proof

By the provenance transition rule:

```text
P_(i+1) = P_i || <p_i>.
```

Sequence append preserves every element of the prefix exactly.

No R_SAT rule has an authorized effect that truncates, replaces, or edits `P_i`.

Any proposed `delta_i^P` not equal to the canonical append is rejected.

Therefore prior provenance cannot be erased or mutated.

QED.

---

# 12. Lemma 2: An assertion cannot lose its introducing lineage

**Lemma.**

Every accepted ledger assertion permanently retains a reconstructible path to its introducing rule, transition, and witness set.

### Proof

Every accepted assertion:

```text
ASSERT(r,mu_i)
```

must satisfy:

```text
mu_i.pid = pid_i.
```

The canonical record `p_i` stores:

```text
step_i
from_i
to_i
rule_i
params_i
witness_refs_i
effect_refs_i.
```

The asserted relation `r` must occur in `effect_refs_i`.

By Lemma 1, `p_i` remains unchanged in every later provenance state.

Therefore the relation's introducing rule, graph transition, and validated witness lineage remain reconstructible for the rest of the trajectory.

QED.

---

# 13. Lemma 3: A later transition cannot forge earlier lineage

**Lemma.**

No later accepted transition can create a provenance record that falsely claims to be the introducing record for an earlier assertion.

### Proof

Let assertion `r` be introduced at transition `i`.

Its ledger metadata contains:

```text
pid_i = i.
```

A later transition `k > i` has canonical provenance identifier:

```text
pid_k = k.
```

The verifier constructs that identifier from the actual transition index.

It does not accept a certificate-selected provenance identifier.

Therefore transition `k` cannot produce a second canonical record with identifier `i`.

Additionally, `P_i` is an immutable prefix, so the original `p_i` cannot be overwritten.

Any later record may reference `p_i` as a parent only after the verifier confirms that the current rule actually depends on the effect introduced by `p_i`.

Thus later transitions may extend lineage but cannot impersonate or replace earlier lineage.

QED.

---

# 14. Lemma 4: A transition cannot forge witness ancestry

**Lemma.**

An accepted transition cannot claim witness ancestry that was not actually validated by its rule application.

### Proof

The canonical provenance record is generated only after RuleOK validates the transition.

For every witness reference, the verifier checks:

```text
WitnessExists(ref)
and
WitnessUsedByRule(ref,rule_i)
and
WitnessValid(ref,S_i).
```

For parent provenance references it additionally checks:

```text
parent_ref < i
and
parent_ref identifies an existing record in P_i.
```

The verifier then derives:

```text
witness_refs_i
parent_refs_i
```

from those validated dependencies.

A certificate-supplied lineage claim not matching the verifier-derived set is rejected.

Therefore an accepted transition cannot invent witness ancestry.

QED.

---

# 15. Lemma 5: Every accepted transition is provenance-monotone

**Lemma.**

For every accepted RTP_SAT transition:

```text
P_i <=_P P_(i+1).
```

### Proof

By Lemma 1, all prior provenance records remain unchanged.

The transition adds one canonical record `p_i`.

By Lemmas 2-4, the new record's effects and dependencies are bound to the actual accepted rule, graph transition, and validated witnesses.

Hence no information recoverable from `P_i` is lost, while new valid lineage is added.

This is exactly the intended meaning of:

```text
P_i <=_P P_(i+1).
```

QED.

---

# 16. Trajectory-level theorem

**Theorem.**

For every accepted RTP_SAT trajectory:

```text
q_0 => q_1 => ... => q_m,
```

provenance is monotone:

```text
P_0 <=_P P_1 <=_P ... <=_P P_m.
```

### Proof

Apply Lemma 5 independently to every accepted transition.

By transitivity of the provenance-preservation relation:

```text
P_h <=_P P_k
```

for all:

```text
0 <= h <= k <= m.
```

Therefore every assertion introduced anywhere on the trajectory retains its complete introducing lineage through the terminal state.

QED.

---

# 17. Theorem PO-10: Provenance Monotonicity

**Theorem.**

The constructed RTP_SAT certificate satisfies provenance persistence throughout every accepted trajectory.

Specifically:

1. every accepted transition preserves all prior provenance;
2. every assertion retains its introducing rule, transition, and validated witness lineage;
3. no later transition can erase an earlier record;
4. no later transition can overwrite or impersonate an earlier provenance identifier;
5. no accepted transition can forge witness ancestry;
6. later records may reference earlier provenance only as validated dependencies.

Therefore:

```text
for every accepted transition i:
P_i <=_P P_(i+1)
```

and:

```text
PO-10 Provenance Monotonicity = PASS.
```

QED.

---

# 18. No new primitive requirement

This proof adds no new Anchor primitive.

It formalizes the existing components already present in RTP:

```text
delta_i^P
P_i
P_i <=_P P_(i+1)
omega_i
RuleOK
AuthorizedEffect
```

The only refinement is that accepted provenance is canonical and verifier-derived rather than arbitrary certificate-authored metadata.

Thus the proof strengthens the existing RTP provenance requirement without changing the problem's expressive class.

---

# 19. Proof-obligation status

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
PO-10 Provenance Monotonicity       PASS
PO-11 Certificate Bounds            OPEN
PO-12 Polynomial Construction       OPEN
```

The remaining proof obligations concern only canonical encoding and polynomial size/construction bounds.
