# Exact RTP Certificate Constant: c_RTP = 8

## Status

Canonical bit-count proof for the bounded Resolution Transition Problem under:

```text
RTP-ENC-v1
RTP-RULES-v1
RTP-BOUNDARIES-v1
```

This document derives the fixed certificate constant:

```text
c_RTP = 8
```

without asymptotic hidden constants.

Let:

```text
N = |RTP-ENC-v1(I)|
```

for a canonical RTP instance `I`.

Canonical RTP requires:

```text
m <= N
|delta_i| <= N
|omega_i| <= N
```

for every certificate transition.

The v1 registries guarantee that all transition-specific certificate material is contained in `delta_i` and `omega_i`. Rule schemas, boundary schemas, relation policies, and reconciliation requirements are instance data; canonical provenance is verifier-derived. There is no additional unbounded per-transition certificate channel.

## 1. Fixed-width primitive

For a given instance length `N >= 1`, define:

```text
FW_N(x)
```

as the unsigned binary representation of integer `x`, left-padded with zeroes to exactly `N` bits.

This is defined for every integer used below because:

```text
0 <= m <= N
0 <= |delta_i| <= N
0 <= |omega_i| <= N
0 <= vertex_id < |V| <= N
```

and for every `N >= 1`:

```text
N <= 2^N - 1.
```

Thus every required count, block length, and vertex identifier fits in one `N`-bit fixed-width field.

## 2. Padded bounded block

For any bit string `x` with:

```text
|x| <= N,
```

define:

```text
PAD_N(x)
```

as `x` followed by zero padding to exactly `N` bits.

Its true semantic length is carried separately in:

```text
FW_N(|x|).
```

Therefore one canonical bounded block is:

```text
BLOCK_N(x)
=
FW_N(|x|) || PAD_N(x)
```

and has exact length:

```text
|BLOCK_N(x)| = 2N.
```

## 3. Canonical certificate serialization

The certificate version is inherited from the canonical instance/version tuple, so it need not repeat the registry names.

Encode:

```text
C = <pi, Delta, Omega>
```

relative to instance `I` as:

```text
Enc_I(C) =
CERT_TAG
||
FW_N(m)
||
FW_N(v_0) || ... || FW_N(v_m)
||
BLOCK_N(delta_0) || ... || BLOCK_N(delta_(m-1))
||
BLOCK_N(omega_0) || ... || BLOCK_N(omega_(m-1))
```

where:

```text
|CERT_TAG| = 1.
```

The fixed tag identifies the canonical certificate record in the decoding context supplied by `I`.

No other delimiters are needed:

- `m` determines the number of trajectory vertices and transition blocks;
- every trajectory identifier is exactly `N` bits;
- every delta/witness block is exactly `2N` bits;
- each bounded block contains its true payload length.

Thus the encoding is uniquely parseable.

## 4. Exact bit count

The components have exact sizes:

### Certificate tag

```text
1 bit.
```

### Transition count

```text
|FW_N(m)| = N.
```

### Trajectory

There are:

```text
m + 1
```

vertex identifiers, each exactly `N` bits:

```text
|pi|_enc = (m+1)N.
```

### Delta sequence

There are `m` delta blocks, each exactly `2N` bits:

```text
|Delta|_enc = 2mN.
```

### Witness sequence

There are `m` witness blocks, each exactly `2N` bits:

```text
|Omega|_enc = 2mN.
```

Therefore:

```text
|Enc_I(C)|
=
1
+ N
+ (m+1)N
+ 2mN
+ 2mN
```

so:

```text
|Enc_I(C)|
=
1 + (5m+2)N.
```

This is an exact equality for the canonical padded representation.

## 5. Apply the trajectory bound

Because:

```text
m <= N,
```

we obtain:

```text
|Enc_I(C)|
<=
1 + (5N+2)N
=
5N^2 + 2N + 1.
```

Now compare this with:

```text
8N^2.
```

For every `N >= 1`:

```text
8N^2 - (5N^2 + 2N + 1)
=
3N^2 - 2N - 1
=
(3N+1)(N-1)
>=
0.
```

Therefore:

```text
|Enc_I(C)| <= 8N^2
```

for every canonical RTP instance.

No finite-exception argument is required.

## 6. Why the transition registries make the count complete

The exact bound depends on there being no hidden certificate payload beyond `Delta` and `Omega`.

Under `RTP-RULES-v1` and `RTP-BOUNDARIES-v1`:

- rule descriptors are encoded in `I`;
- boundary descriptors are encoded in `I`;
- relation-policy tables are encoded in `I`;
- reconciliation requirements are encoded in `I`;
- rule/boundary semantics are fixed by immutable registries;
- state/relation/boundary mutations are contained in `delta_i`;
- transition witnesses and authorization references are contained in `omega_i`;
- accepted provenance is verifier-derived from the accepted transition and bounded references.

Thus the certificate has exactly three semantic components:

```text
pi
Delta
Omega
```

plus fixed framing already counted above.

## 7. Exact certificate theorem

**Theorem.**

For every canonical RTP instance `I` with:

```text
N = |RTP-ENC-v1(I)|,
```

and every syntactically valid canonical RTP certificate satisfying:

```text
m <= N
|delta_i| <= N
|omega_i| <= N,
```

the canonical serialization satisfies:

```text
|Enc_I(C)| <= 8N^2.
```

Hence the fixed language constant may be chosen as:

```text
c_RTP = 8.
```

QED.

## 8. Stronger bound

The proof actually establishes the stronger exact envelope:

```text
|Enc_I(C)|
<=
5N^2 + 2N + 1.
```

The language uses:

```text
8N^2
```

because it is a simple uniform quadratic ceiling valid for every `N >= 1`.

The value `8` is therefore an exact proved language constant, though it is not claimed to be the minimal possible constant.
