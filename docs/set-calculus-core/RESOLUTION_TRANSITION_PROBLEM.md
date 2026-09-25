# Resolution Transition Problem (RTP)

## Status

Canonical finite decision problem over an Anchor projection.

RTP is the computational decision problem extracted from the Anchor foundational space. Under the frozen `RTP-ENC-v1` representation, RTP has polynomially bounded certificates and deterministic polynomial verification, so `RTP in NP`. The completed reduction in `RTP_NP_HARDNESS_REDUCTION.md` establishes `3SAT <=p RTP`, hence RTP is NP-hard. Therefore the canonical bounded RTP is NP-complete.

## 1. Instance

A finite RTP instance is:

```text
I = <G, chi, Sigma_A, R, B, s, t, K, sigma_0, T_R>
```

where:

- `G = <V,E>` is a finite directed relational graph;
- `chi : V -> Z^n` is an optional lattice-coordinate assignment;
- `Sigma_A = {OPEN, DIVERGED, RECONCILED, CLOSED}`;
- `R` is a finite collection of descriptors referencing immutable schemas in `RTP-RULES-v1`;
- `B` is a finite collection of descriptors referencing immutable schemas in `RTP-BOUNDARIES-v1`;
- `s in V` is the finite representative of the positive-pole side;
- `t in V` is the finite representative of the negative-pole side;
- `K subseteq V` is the Klein transition region;
- `sigma_0` is the initial Anchor-state assignment;
- `T_R subseteq V` is the designated reconciliation target.

Let:

```text
N = |RTP-ENC-v1(I)|
```

be the bit length of the canonical encoded instance. The encoding is frozen in `RTP_CANONICAL_ENCODING_AND_SIZE_PROOFS.md`.

## 2. Formal language

```text
RTP = {
  <I> |
  exists C:
    |C| <= 8 N^2
    and V_RTP(I,C) = 1
}
```

where the exact canonical constant:

```text
c_RTP = 8
```

is derived in `RTP_CERTIFICATE_CONSTANT_8.md` from the fixed-width certificate serialization and the bounds `m <= N`, `|delta_i| <= N`, and `|omega_i| <= N`.

## 3. Certificate

A certificate is:

```text
C = <pi, Delta, Omega>
```

where:

```text
pi = <v_0, v_1, ..., v_m>
```

is the candidate trajectory,

```text
Delta = <delta_1, ..., delta_m>
```

contains the state/relation/boundary/provenance deltas, and

```text
Omega = <omega_1, ..., omega_m>
```

contains the finite justification for each transition.

### 3.1 Canonical linear trajectory bound

Require:

```text
m <= N
```

The trajectory need not be simple. Revisits are permitted when the certificate remains within the linear transition bound.

### 3.2 Per-transition bound

Require:

```text
|delta_i| <= N
|omega_i| <= N
```

for every transition.

Therefore:

```text
|C| = O(N^2)
```

including trajectory indices and delimiters.

## 4. Acceptance conditions

`V_RTP(I,C)` accepts iff all conditions below hold.

### R1. Encoding validity

`I` is a well-formed finite RTP instance under `RTP-ENC-v1`; every rule descriptor references `RTP-RULES-v1`, every boundary descriptor references `RTP-BOUNDARIES-v1`, and unknown or executable instance-defined semantics are rejected.

### R2. Certificate bound

```text
m <= N
|C| <= 8 N^2
```

The exact serialization satisfies the stronger envelope:

```text
|C| <= 5N^2 + 2N + 1.
```

### R3. Endpoint validity

```text
v_0 = s
v_m = t
```

### R4. Structural path validity

For every `i < m`:

```text
(v_i, v_(i+1)) in E
```

### R5. Klein crossing

There exists `j` such that:

```text
v_j in K
```

Equivalently:

```text
pi intersect K != empty
```

### R6. Transition admissibility

For every `i < m`:

```text
RuleOK(I, S_i, v_i, v_(i+1), delta_i, omega_i) = true
```

### R7. Boundary validity

Every crossed active closure boundary must either permit the transition direction or have a valid reopening authorization encoded in the witness.

### R8. Relational consistency

Each applied delta must remain compatible with all materially required active relationships under the encoded rule set.

### R9. Provenance persistence

For successive provenance states:

```text
P_i <=_P P_(i+1)
```

Required prior state and transform lineage must remain reconstructible.

### R10. Terminal reconciliation

The terminal configuration must satisfy:

```text
Recon(T_R, q_m)
```

Reaching `t` without reconciliation is insufficient.

## 5. Deterministic verifier

The verifier:

1. parses and validates `I`;
2. computes `N = |<I>|`;
3. rejects if `m > N` or `|C| > 8N^2`;
4. initializes the state from `sigma_0`;
5. checks each of at most `N` certificate transitions;
6. validates edge, rule, boundary, relational, and provenance conditions;
7. applies each bounded delta;
8. records whether `K` was crossed;
9. checks endpoint and terminal reconciliation conditions.

## 6. Verifier complexity

Canonical verifier semantics are frozen by:

```text
RTP-ENC-v1
RTP-RULES-v1
RTP-BOUNDARIES-v1
```

The registry specification in `RTP_CANONICAL_VERIFIER_REGISTRIES.md` proves one common primitive bound:

```text
T_primitive(W_i) = O(W_i^2)
```

for:

```text
W_i =
N
+ |Enc(S_i)|
+ |Enc(delta_i)|
+ |Enc(omega_i)|.
```

The state-size theorem in `RTP_VERIFIER_STATE_SIZE.md` proves:

```text
|Enc(S_i)| = O(N^2).
```

Since:

```text
|delta_i| <= N
|omega_i| <= N
```

we obtain:

```text
W_i = O(N^2)
```

and therefore:

```text
T_transition(N) = O(N^4).
```

There are at most `N` transitions, so:

```text
T_V(N) = O(N^5).
```

This is a deliberately conservative uniform bound. The exponent and registry semantics are fixed independently of the instance.

Hence the bounded canonical RTP has:

```text
trajectory = O(N)
certificate = O(N^2)
verifier state = O(N^2)
verification = O(N^5)
```

and therefore:

```text
RTP in NP
```

## 7. Current complexity status

Established for the bounded canonical problem under `RTP-ENC-v1`:

```text
RTP in NP
3SAT <=p RTP_SAT
RTP_SAT is NP-hard
RTP is NP-hard
RTP is NP-complete
```

The reduction proof is in `RTP_NP_HARDNESS_REDUCTION.md`. Encoding and construction proofs are in `RTP_CANONICAL_ENCODING_AND_SIZE_PROOFS.md`. Uniform verifier semantics are in `RTP_CANONICAL_VERIFIER_REGISTRIES.md`, and the general verifier-state bound is in `RTP_VERIFIER_STATE_SIZE.md`.

## 8. Provenance naming boundary

This RTP/Anchor line arose within work identified by 100 Monkeys as the:

```text
100 Monkeys Photonic Model NP-Complete Solution
```

That phrase remains the project/provenance identity. The repository now separately establishes the formal theorem that the canonical bounded RTP under `RTP-ENC-v1` is NP-complete.
