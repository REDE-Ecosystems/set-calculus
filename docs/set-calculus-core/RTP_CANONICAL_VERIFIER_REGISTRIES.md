# RTP-RULES-v1 and RTP-BOUNDARIES-v1

## Status

Canonical immutable verifier registries for the bounded Resolution Transition Problem under `RTP-ENC-v1`.

A canonical RTP instance contains finite data and references to fixed registry schemas. It does not contain executable verifier logic.

The version tuple:

```text
<RTP-ENC-v1, RTP-RULES-v1, RTP-BOUNDARIES-v1>
```

defines one immutable language version.

Any semantic change requires a new version identifier.

## 1. Version immutability

For v1, the following are immutable:

- schema identifiers;
- schema preconditions;
- authorized effects;
- witness semantics;
- boundary semantics;
- reopening semantics;
- canonical predicates;
- runtime contract.

No schema may be added to v1 after publication.

Unknown registry identifiers are malformed canonical encodings and are rejected by `DecodeOK`.

Later versions do not retroactively change the meaning of v1 instances.

## 2. Canonical rule descriptors

A rule descriptor is finite data:

```text
RuleDescriptor =
<
  descriptor_id,
  schema_id,
  target_refs,
  required_active,
  required_inactive,
  allowed_from_states,
  relation_effect,
  state_effect,
  boundary_ref,
  witness_refs
>
```

Every reference names an explicitly encoded instance object.

No field may contain executable code, bytecode, callbacks, recursive predicates, external calls, or arbitrary logical programs.

Rule guards are finite conjunctions of explicit membership/state tests.

## 3. RTP-RULES-v1

The registry contains exactly seven schemas:

```text
RR1 GUARD
RR2 ASSERT_RELATION
RR3 RETRACT_RELATION
RR4 SET_STATE
RR5 ASSERT_AND_SET_STATE
RR6 ACTIVATE_BOUNDARY
RR7 REOPEN_BOUNDARY
```

Thus:

```text
|RTP-RULES-v1| = 7.
```

### RR1 GUARD

Checks explicit guards and permits no state, relation, or boundary effect.

### RR2 ASSERT_RELATION

If guards hold and `r in Rel(I)`, authorizes exactly:

```text
ASSERT(r,mu_i)
```

with canonical current-transition provenance.

### RR3 RETRACT_RELATION

If guards hold, `r` is active, and `Retractable(r)`, authorizes exactly:

```text
RETRACT(r,mu_i)
```

A persistent relation may not be retracted.

### RR4 SET_STATE

If guards hold and the current state is listed in `allowed_from_states`, authorizes one explicit transition:

```text
sigma_A(x) := s'
```

with `s' in Sigma_A`.

### RR5 ASSERT_AND_SET_STATE

Atomically authorizes one relation assertion and one state transition:

```text
ASSERT(r,mu_i)
sigma_A(x) := s'
```

No additional effect is permitted.

### RR6 ACTIVATE_BOUNDARY

Activates one boundary already encoded in the instance:

```text
ACTIVATE(b)
```

The boundary definition itself is never dynamically generated.

### RR7 REOPEN_BOUNDARY

For active boundary `b`, authorizes:

```text
REOPEN(b)
```

iff `ReopenOK(b,S_i,omega_i)` is true.

Reopening changes the active-boundary view and preserves closure/reopening provenance.

## 4. Relation policy forms

The only v1 relation-policy forms are:

```text
PERSISTENT(r)
RETRACTABLE(r)
MUTEX(r_1,r_2)
REQUIRES(r_1,r_2)
```

`RelationOK` checks the proposed post-transition active view against all explicitly encoded policy records.

No recursive or transitive semantic closure is materialized.

## 5. RTP-BOUNDARIES-v1

A boundary descriptor is finite data:

```text
BoundaryDescriptor =
<
  boundary_id,
  schema_id,
  scope_edges,
  authority_refs,
  required_active,
  required_inactive,
  required_states,
  closure_metadata
>
```

The registry contains exactly four schemas:

```text
RB1 HARD_CLOSURE
RB2 RELATIONAL_REOPEN
RB3 AUTHORITY_REOPEN
RB4 CONJUNCTIVE_REOPEN
```

Thus:

```text
|RTP-BOUNDARIES-v1| = 4.
```

### RB1 HARD_CLOSURE

An active RB1 boundary rejects every transition on its explicitly listed ordered `scope_edges`. It cannot be reopened.

### RB2 RELATIONAL_REOPEN

May reopen iff all explicit required-active relations are active and all explicit required-inactive relations are inactive.

### RB3 AUTHORITY_REOPEN

May reopen iff the witness references one explicitly encoded authorized identifier and that authority/provenance reference is valid.

The verifier never infers authority from names, roles, topology, or external sources.

### RB4 CONJUNCTIVE_REOPEN

May reopen iff all explicit relation, state, authority, and provenance requirements hold simultaneously.

No arbitrary expression language exists in v1.

## 6. Canonical predicates

Canonical v1 verification uses only:

```text
DecodeOK
RuleRefOK
GuardsOK
AuthorizedEffectOK
StateOK
RelationOK
ProvOK
ReopenOK
BoundaryOK
ReconOK
RuleOK
```

### StateOK

Requires all state targets to exist, all new states to belong to `Sigma_A`, and all mutations to match the selected registered rule.

### RelationOK

Requires all mentioned relations to belong to `Rel(I)`, all effects to be authorized, persistent relations not to be retracted, and explicit `MUTEX` / `REQUIRES` policies to hold in the prospective active view.

### ProvOK

Uses append-only verifier-derived provenance:

```text
P_(i+1) = P_i || <CanonicalProv_i>
```

with:

```text
pid_i = i
parent_ref < i.
```

Prior provenance is never rewritten.

### ReopenOK

Dispatches only on RB1-RB4 and evaluates their finite explicit guards.

### BoundaryOK

Checks each active boundary whose explicit ordered scope contains the proposed edge. A guarded edge is rejected unless the boundary is legally reopened by RR7 before crossing.

### ReconOK

Each target uses only explicit requirements:

```text
STATE(target,state)
ALL {r_1,...,r_k}
ANY {r_1,...,r_k}
```

For every target `x in T_R`, v1 requires:

```text
sigma_A(x) = RECONCILED
```

plus satisfaction of all encoded `ALL` / `ANY` groups.

No search for a reconciliation proof is performed.

### RuleOK

```text
RuleOK
=
RuleRefOK
and GuardsOK
and AuthorizedEffectOK
and StateOK
and RelationOK
and BoundaryMutationOK
and ProvOK
```

## 7. No succinct-expansion rule

A rule, boundary, witness, relation, or derived view may reference encoded objects but may not require materialization of an object whose explicit representation is superpolynomial in the current explicit working representation.

For v1, every object materialized by the verifier must have encoded size at most the current working-size parameter.

Derived views such as `Active(Lambda)` may be indexed or evaluated lazily. They may not materialize an implicit exponential closure.

## 8. Working-size parameter

For transition `i` define:

```text
W_i =
|Enc(I)|
+ |Enc(S_i)|
+ |Enc(delta_i)|
+ |Enc(omega_i)|.
```

All v1 primitive runtime guarantees use `W_i`.

## 9. Common polynomial runtime bound

There exist fixed registry constants `a>0` and `b>=0` such that every v1 canonical predicate terminates within:

```text
a W_i^2 + b
```

deterministic steps.

Hence:

```text
T_primitive(W_i) = O(W_i^2).
```

Reason: every v1 predicate performs only finite parsing, explicit-list scans, identifier comparisons, finite membership checks, fixed state checks, explicit provenance-reference checks, and finite ALL/ANY or conjunction evaluation.

No v1 predicate performs:

- unbounded recursion;
- transitive-closure materialization;
- satisfiability search;
- theorem proving;
- fixed-point iteration;
- executable user predicates;
- external calls;
- superpolynomial decompression.

Because the registries are finite and immutable, the exponent and maximum constants are fixed properties of the language, not of the instance.

## 10. Uniformity theorem

For every legal canonical RTP transition under v1:

```text
T_registry(W_i) = O(W_i^2)
```

with one common exponent and instance-independent constants.

Thus the registry semantics satisfy the uniformity condition required by the general RTP membership-in-NP proof.

## 11. RTP_SAT compatibility

The SAT reduction embeds into the canonical schemas:

```text
ChooseTrue               -> RR2
ChooseFalse              -> RR2
PositiveLiteralValidation-> RR1
NegativeLiteralValidation-> RR1
ClauseReconcile          -> RR5
KleinPass                -> RR1
AssignmentPersistence    -> PERSISTENT relation policy
```

The reduction has no active boundaries.

Therefore its existing verifier semantics are a specialization of the canonical v1 registry semantics.
