# Anchor v0.1 - Foundational Resolution Space

## Status

Foundational Set Calculus research formalization.

The Anchor is the minimal relational substrate required to define resolution-space structure before any particular physical, historical, spiritual, computational, or interpretive model is asserted.

Primary source artifact:

- `docs/set-calculus-core/artifacts/Resolution_Space_Framework.pdf`
- checksum: `docs/set-calculus-core/artifacts/Resolution_Space_Framework.sha256`
- provenance lineage: `docs/provenance/PHOTONIC_MODEL_ANCHOR_LINEAGE.md`

The source framework defines a research space rather than an accuracy claim. The Anchor preserves that boundary.

## 1. Layer boundary

```text
Anchor
  -> relational resolution substrate
  -> Set Calculus operations / evidence
  -> models and specializations
  -> hypotheses and experiments
```

The Anchor does not make the source framework's historical or physical hypotheses foundational axioms.

### 1.1 State vocabulary boundary

The source artifact uses:

```text
OPEN
DIVERGED
RECONCILED
CLOSED
```

These are defined here as Anchor reconciliation states.

They are distinct from the current Set Calculus public resolution evidence states:

```text
UNRESOLVED
PARTIAL
VALID
INVALID
```

No automatic identity mapping between these two state systems is asserted.

```text
Anchor reconciliation state
!=
Set Calculus public resolution status
```

Any bridge between them must be explicit and provenance-preserving.

## 2. Canonical signature

Define an Anchor:

```text
A = <X, E, Omega+, Omega-, K, Sigma_A, B, P, T, Adm>
```

where:

- `X` is the set of resolution positions;
- `E` is the relational structure;
- `Omega+` and `Omega-` are framing poles;
- `K` is the Klein transition region;
- `Sigma_A` is the Anchor reconciliation-state alphabet;
- `B` is the closure-boundary domain;
- `P` is the provenance domain;
- `T` is the transition relation;
- `Adm` is the transition-admissibility predicate.

## 3. Primitive objects

### 3.1 Resolution positions

`X` is the set of distinguishable positions in a resolution space.

For `x in X`, `x` is a resolution position.

A position may represent a state, proposition, relationship, observation, configuration, possibility, or another distinguishable location in the resolution structure.

When coordinates are useful:

```text
chi : X -> D^n
```

may assign positions to an n-dimensional coordinate domain.

### 3.2 Relations

```text
E subseteq X x X
```

is the set of explicit relationships between resolution positions.

Write `x -> y` when `(x,y) in E`.

A traversal edge may be directed without requiring resolution itself to be historically or temporally one-directional.

Define:

```text
N(x) = { y in X | (x,y) in E or (y,x) in E }
```

as the relational neighborhood of `x`.

### 3.3 Framing poles

Introduce distinguished formal boundary markers:

```text
Omega+
Omega-
```

They are boundary objects rather than ordinary resolvable positions:

```text
Omega+, Omega- notin X
```

Within the source model:

- `Omega+` frames the immediate/easily resolved outlier extreme;
- `Omega-` frames the non-terminating/unresolvable outlier extreme.

The poles are relational markers. They are not themselves successful resolutions.

### 3.4 Klein transition region

```text
K subseteq X
```

is the set of positions designated as the Klein/origin transition region.

A completed monadic resolution must intersect `K`.

## 4. Monad

Define the foundational monadic structure:

```text
M = <Omega+, K, Omega->
```

The triple is treated as one relational object rather than three independently complete spaces.

```text
Omega+ <-> K <-> Omega-
```

The meaning of the middle is relationally defined by the framing poles, and the poles derive their role by relation to the middle.

## 5. Anchor reconciliation states

Define:

```text
Sigma_A = { O, D, R, C }
```

where:

```text
O = OPEN
D = DIVERGED
R = RECONCILED
C = CLOSED
```

An Anchor state assignment is:

```text
sigma_A : X -> Sigma_A
```

Interpretation:

- `OPEN`: relationships remain available for continued reconciliation;
- `DIVERGED`: the carried local resolution has not reconciled all currently available required relationships;
- `RECONCILED`: currently required available relationships have been incorporated;
- `CLOSED`: an explicit boundary limits further reconciliation across part of the structure.

## 6. Configuration

A configuration is:

```text
q = <x, sigma_A, A_E, B_A, P_q>
```

where:

- `x in X` is the current resolution position;
- `sigma_A` is the current Anchor state assignment;
- `A_E subseteq E` is the active relational structure;
- `B_A subseteq B` is the active closure set;
- `P_q in P` is the accumulated provenance state.

Let `Q` denote the set of valid configurations.

## 7. Transition relation

Define:

```text
T subseteq Q x Q
```

Write:

```text
q_i => q_(i+1)
```

when one admissible resolution operation transforms `q_i` into `q_(i+1)`.

A transition may alter position, reconciliation state, active relationships, closures, and provenance.

It may also alter the interpretation of earlier positions without deleting their provenance.

## 8. Trajectory

A finite Anchor trajectory is:

```text
pi = <q_0, q_1, ..., q_m>
```

such that:

```text
q_i => q_(i+1)
```

for every `0 <= i < m`.

A trajectory is not required to be a simple path. Revisiting a position or configuration is permitted when required by divergence, backward reconciliation, newly available relationships, or reopening.

This is consistent with the existing Set Calculus distinction:

```text
trajectory length != resolution depth
```

## 9. Admissibility

Introduce:

```text
Adm(q_i, q_(i+1))
```

which is true exactly when the proposed transition satisfies the active rules governing that portion of the Anchor.

Therefore:

```text
q_i => q_(i+1)
```

requires:

```text
Adm(q_i, q_(i+1)) = true
```

The Anchor predicate is a substrate-level predicate. A Set Calculus implementation may refine it with Context, Authority, Identity, Invariant, State, and Provenance witness semantics.

## 10. Closure boundary

A closure boundary is:

```text
b = <X_b, D_b, A_b, Q_b, S_b>
```

where:

- `X_b` identifies the bounded relationship or region;
- `D_b` identifies the bounded direction or directions;
- `A_b` records the authority or rule establishing closure;
- `Q_b` defines conditions permitting reopening;
- `S_b` records the state at closure.

Closure constrains reconciliation propagation. It does not erase the underlying relation or its provenance.

## 11. Provenance

Let `P` be the provenance domain.

Define a provenance partial order `<=_P` where:

```text
P_i <=_P P_(i+1)
```

means all materially required information recoverable from `P_i` remains recoverable from `P_(i+1)`.

Interpretation may change. Required history may not silently disappear.

## 12. Foundational axioms

### A1. Monadic unity

```text
M = <Omega+, K, Omega->
```

is one relational object. No member of the triple is treated as a complete resolution structure independently of the other two.

### A2. Relational resolution

Resolution propagates through relations rather than being restricted to historical succession.

```text
historical order != resolution direction
```

Unless bounded by an active closure, later relational information may alter the interpretation of earlier positions, while earlier positions constrain later admissible states.

### A3. Explicit reconciliation state

Every active Anchor resolution position has an explicit reconciliation state:

```text
sigma_A(x) in { OPEN, DIVERGED, RECONCILED, CLOSED }
```

### A4. Reconciliation-state mutability

Anchor reconciliation state is mutable when admissible relational evidence changes.

Permitted transition families include:

```text
DIVERGED -> RECONCILED
RECONCILED -> DIVERGED
CLOSED -> OPEN
```

when the governing rules authorize them.

These are families, not a complete transition table.

### A5. Divergence is not contradiction

```text
DIVERGED != INVALID
DIVERGED != contradiction
```

A divergent configuration remains structurally active and may later reconcile.

### A6. Closure bounds propagation

```text
Closure => Boundary(Propagation)
```

not:

```text
Closure => Delete(Relation)
```

### A7. Provenance persistence

For every admissible transition:

```text
q_i => q_(i+1)
```

require:

```text
P_i <=_P P_(i+1)
```

### A8. Interface resolution

For a completed monadic resolution trajectory `pi`:

```text
pi intersect K != empty
```

The framing poles do not themselves constitute resolution.

### A9. Admissible resolution

A trajectory is valid only when every constituent transition is admissible.

```text
Valid(pi)
iff
for all i < m: Adm(q_i, q_(i+1))
```

Arrival at a desired endpoint does not retroactively validate an inadmissible trajectory.

### A10. Reconciliation

A target `T_R subseteq X` is reconciled in terminal configuration `q_m` iff the required currently available relations have been incorporated under the active rule set.

```text
Recon(T_R, q_m)
iff
sigma_A(T_R) = RECONCILED
and
RequiredRelations(T_R) subseteq Incorporated(q_m)
```

## 13. Derived resolution predicate

Define:

```text
Resolve_A(q_0, T_R)
```

iff there exists a finite trajectory:

```text
pi = <q_0, ..., q_m>
```

such that:

```text
Valid(pi)
and Cross_K(pi)
and PreserveProvenance(pi)
and RespectClosure(pi)
and Recon(T_R, q_m)
```

## 14. Computational projection

Complexity theory is applied only after an Anchor problem has been projected into a finite encoded instance.

The canonical Resolution Transition Problem is defined in `RESOLUTION_TRANSITION_PROBLEM.md`.

For that bounded canonical problem under:

```text
RTP-ENC-v1
RTP-RULES-v1
RTP-BOUNDARIES-v1
```

the canonical verifier proofs establish:

```text
trajectory transitions <= N
certificate size = O(N^2)
verifier state size = O(N^2)
primitive registry check = O(W^2)
verification time = O(N^5)
```

under the deliberately conservative uniform verifier model defined in `RTP_CANONICAL_VERIFIER_REGISTRIES.md` and `RTP_VERIFIER_STATE_SIZE.md`.

Therefore the bounded canonical RTP is in NP.

This is not, by itself, a proof of NP-hardness or NP-completeness.

## 15. Non-primitive structures

The following may inhabit an Anchor without being required to define every Anchor:

- the 27 internal coordinates described by the source framework;
- Self;
- Entropy;
- specific physical interpretations;
- specific historical interpretations;
- specific spiritual interpretations;
- cycle lengths;
- observer models;
- application hypotheses.

The source framework's 27 + Self + Entropy structure is therefore treated as an Anchor instantiation rather than as a universal Anchor axiom.

## 16. Minimal invariant core

```text
RELATION
   |
STATE
   |
TRANSITION
   |
ADMISSIBILITY
   |
KLEIN INTERFACE
   |
RECONCILIATION
```

under persistent constraints:

```text
CLOSURE
PROVENANCE
```

Canonical object:

```text
A = <X, E, Omega+, Omega-, K, Sigma_A, B, P, T, Adm>
```

The Anchor defines the space in which resolution can be represented.

Set Calculus defines the formal operations, evidence, transforms, and public resolution classifications applied over that space.
