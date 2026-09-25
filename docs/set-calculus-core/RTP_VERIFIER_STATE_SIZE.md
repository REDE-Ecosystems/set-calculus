# RTP Verifier State-Size Theorem

## Status

Canonical state-size proof for the bounded Resolution Transition Problem under:

```text
RTP-ENC-v1
RTP-RULES-v1
RTP-BOUNDARIES-v1
```

This document proves that every explicit verifier state occurring during verification has polynomial size and, under the canonical certificate bounds, satisfies:

```text
|Enc(S_i)| = O(N^2)
```

where:

```text
N = |RTP-ENC-v1(I)|.
```

This closes the intermediate-state-size assumption identified in the RTP membership-in-NP audit.

## 1. Canonical verifier state

At transition index `i`, use:

```text
S_i =
<
  sigma_i,
  Lambda_i,
  B_i,
  P_i
>
```

where:

- `sigma_i` is the explicit Anchor reconciliation-state assignment;
- `Lambda_i` is the append-only Active Relational Ledger;
- `B_i` is the active-boundary view;
- `P_i` is append-only canonical provenance.

The current trajectory vertex `v_i` is stored in the certificate trajectory and need not be duplicated in `S_i`.

Derived indexes and caches are not semantic state. They may be omitted, rebuilt, or maintained only within the no-succinct-expansion rule.

## 2. Initial-state bound

The initial state is determined entirely by the encoded instance:

```text
sigma_0
Lambda_0
B_0
P_0.
```

Therefore:

```text
|Enc(S_0)| = O(N).
```

In particular:

- `sigma_0` has at most one finite state value per explicitly encoded vertex or an encoded default plus explicit overrides;
- initial active relations and boundaries must be explicitly represented by the instance;
- initial provenance, if any, must be explicitly encoded.

No initial semantic object may be larger than a polynomial function of its own explicit input encoding.

Under v1 the initial explicit state is bounded linearly by the instance representation.

## 3. Certificate bounds

Canonical RTP requires:

```text
m <= N
|delta_i| <= N
|omega_i| <= N
```

for every transition.

Thus:

```text
sum_(h < i) |delta_h|
<=
iN
<=
N^2.
```

The state-size proof uses this accumulated-delta budget.

## 4. Reconciliation-state component

`sigma_i` maps explicitly encoded vertices to one of four states:

```text
OPEN
DIVERGED
RECONCILED
CLOSED.
```

No transition may create a new vertex.

Therefore the number of state slots never exceeds:

```text
|V| <= N.
```

Using a dense array ordered by canonical vertex identifier, each state uses constant space.

Hence:

```text
|Enc(sigma_i)| = O(N).
```

Even with explicit identifier/value pairs, the component remains polynomial and below the final O(N^2) bound.

## 5. Active Relational Ledger component

The ledger evolves only by append:

```text
Lambda_(h+1)
=
Lambda_h || delta_h^Lambda.
```

Every serialized ledger event is contained in the corresponding transition delta together with only canonical bounded lineage references.

For each transition, the total new relation-event payload is at most:

```text
O(|delta_h| + |omega_h| + log N)
=
O(N).
```

The canonical registry forbids a ledger event from materializing an implicit larger object.

Therefore after `i` transitions:

```text
|Enc(Lambda_i)|
<=
|Enc(Lambda_0)|
+
O(
  sum_(h<i)
  (|delta_h| + |omega_h| + log N)
)
```

and since `i <= N`:

```text
|Enc(Lambda_i)| = O(N^2).
```

The active relation view:

```text
Active(Lambda_i)
```

is derived from the ledger.

It is not separately materialized as an exponentially expanded closure.

## 6. Active-boundary component

Every boundary belongs to the explicitly encoded finite instance boundary set:

```text
B.
```

No transition may create a new boundary descriptor.

`B_i` records only whether an encoded boundary is currently active.

Therefore:

```text
|B_i| <= |B| <= N
```

in cardinality.

A canonical bit-vector representation ordered by boundary identifier requires:

```text
O(N)
```

bits.

Activation and reopening provenance is stored in `P_i`, not copied recursively into `B_i`.

Thus:

```text
|Enc(B_i)| = O(N).
```

## 7. Provenance component

For every accepted transition:

```text
P_(h+1)
=
P_h || <p_h>
```

where:

```text
p_h = CanonicalProv(...)
```

is verifier-derived.

A canonical provenance record contains only:

- transition index;
- from/to references;
- registered rule reference;
- finite rule parameters;
- witness references;
- parent provenance references;
- effect references.

It does not recursively embed:

- the complete prior provenance sequence;
- the complete prior verifier state;
- the complete witness objects already encoded elsewhere.

All ancestry is represented by backward references:

```text
parent_ref < h.
```

The reference lists used by `p_h` are bounded by the explicitly supplied transition/witness material.

Hence:

```text
|Enc(p_h)|
=
O(|delta_h| + |omega_h| + log N)
=
O(N).
```

There are at most `N` provenance records.

Therefore:

```text
|Enc(P_i)| = O(N^2).
```

## 8. No hidden derived-state expansion

Under `RTP-RULES-v1` and `RTP-BOUNDARIES-v1`, all semantic state is one of:

- explicitly encoded finite state slots;
- append-only ledger records;
- finite active-boundary flags;
- append-only provenance records.

The following are derived views:

```text
Active(Lambda_i)
relation-policy satisfaction
boundary scope membership
reconciliation requirement satisfaction
provenance ancestry lookup
```

They may be computed by scans or polynomial-size indexes.

They are not new semantic objects and may not be materialized as superpolynomial closures.

The registry's no-succinct-expansion rule rejects any semantics that would require such materialization.

## 9. Theorem: verifier state size

**Theorem.**

For every canonical RTP instance `I), every certificate satisfying the canonical size bounds, and every verifier step `0 <= i <= m`:

```text
|Enc(S_i)| = O(N^2).
```

### Proof

By Sections 4-7:

```text
|Enc(sigma_i)|  = O(N)
|Enc(Lambda_i)| = O(N^2)
|Enc(B_i)|      = O(N)
|Enc(P_i)|      = O(N^2).
```

Therefore:

```text
|Enc(S_i)|
=
O(N)
+
O(N^2)
+
O(N)
+
O(N^2)
=
O(N^2).
```

No other semantic state component exists in the canonical v1 verifier.

By Section 8, derived views do not introduce superpolynomial materialization.

Hence:

```text
|Enc(S_i)| = O(N^2).
```

QED.

## 10. Working-size corollary

For transition `i` define:

```text
W_i =
N
+ |Enc(S_i)|
+ |Enc(delta_i)|
+ |Enc(omega_i)|.
```

Using:

```text
|Enc(S_i)| = O(N^2)
|delta_i| <= N
|omega_i| <= N
```

gives:

```text
W_i = O(N^2).
```

## 11. Uniform transition-time corollary

The v1 verifier registries establish:

```text
T_primitive(W_i) = O(W_i^2).
```

Therefore:

```text
T_transition(N)
=
O((N^2)^2)
=
O(N^4).
```

There are at most:

```text
N
```

transitions.

Hence all transition verification costs:

```text
O(N^5).
```

Initial decoding and terminal `ReconOK` are bounded by the same explicit-state/registry model and remain polynomial.

Thus:

```text
T_V(N) = O(N^5)
```

is a valid conservative uniform bound.

## 12. NP-membership consequence

Canonical RTP has:

```text
certificate size = O(N^2)
verification time = O(N^5)
```

under one immutable verifier semantics and one instance-independent exponent.

Therefore:

```text
RTP in NP.
```

This closes the hidden intermediate-state-size and nonuniform-verifier assumptions identified by audit.

## 13. Scope

This theorem applies specifically to the canonical bounded RTP language defined by:

```text
RTP-ENC-v1
RTP-RULES-v1
RTP-BOUNDARIES-v1.
```

A future registry version must establish its own state-size and runtime theorems before inheriting the same complexity classification.
