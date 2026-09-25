# RTP-ENC-v1 Canonical Encoding and Final Size Proofs

## Status

Canonical finite encoding for the Resolution Transition Problem and final size/construction proof artifact for the `3SAT <=p RTP` reduction.

This document freezes:

```text
RTP-ENC-v1
```

and discharges:

```text
PO-1  Canonical Encoding       = PASS
PO-11 Certificate Bounds       = PASS
PO-12 Polynomial Construction  = PASS
```

Together with the previously discharged PO-2 through PO-10, this completes the correctness obligations for the reduction.

---

# 1. Canonical encoded instance

For RTP-ENC-v1, an instance is serialized as a finite tagged binary record:

```text
Enc(I) =
<
  HEADER,
  COUNTS,
  VERTICES,
  EDGES,
  COORDINATES,
  RULES,
  BOUNDARIES,
  ENDPOINTS,
  KLEIN,
  INITIAL_STATE,
  TARGETS
>
```

The encoded instance size is:

```text
N = |Enc(I)|
```

measured in bits.

Every section is length-delimited.

Every vertex and edge is represented explicitly.

No graph element exists only implicitly in executable code.

---

# 2. Primitive encoding forms

RTP-ENC-v1 uses three primitive forms.

## 2.1 Fixed tags

Record and node kinds use a fixed finite tag alphabet.

Examples:

```text
START
ASSIGN_STAGE
TRUE_CHOICE
FALSE_CHOICE
KLEIN
CLAUSE_STAGE
POS_LITERAL
NEG_LITERAL
TERMINAL
```

Every record contains at least one tag bit.

## 2.2 Natural numbers

Natural numbers are encoded by a prefix-free unsigned-integer encoding:

```text
U(k)
```

with:

```text
|U(k)| = O(log(k+2)).
```

## 2.3 Identifiers

Vertices are numbered:

```text
0,1,...,|V|-1.
```

A vertex reference uses the corresponding `U(k)` representation.

Thus every reference requires:

```text
O(log(|V|+2))
```

bits.

---

# 3. RTP-ENC-v1 sections

## 3.1 HEADER

The header contains a fixed magic value and version identifier:

```text
RTP-ENC-v1
```

plus fixed encoding flags.

Its size is constant.

## 3.2 COUNTS

Stores:

```text
|V|
|E|
|B|
|K|
|T_R|
```

as prefix-free integers.

## 3.3 VERTICES

Each vertex has one explicit record:

```text
<VertexType, parameters>
```

For RTP_SAT the fixed vertex label alphabet is:

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

Each vertex therefore consumes at least one encoded bit.

Consequently:

```text
N >= |V|.
```

This lower bound is independent of compression choices for identifiers or parameters because RTP-ENC-v1 requires one explicit nonempty record per vertex.

## 3.4 EDGES

Each edge is encoded explicitly as:

```text
<source-id,target-id>
```

## 3.5 COORDINATES

The section contains either:

```text
ABSENT
```

or a finite explicit coordinate table.

For RTP_SAT:

```text
chi = ABSENT.
```

## 3.6 RULES

The rule section contains identifiers for the fixed rule schemas available to the instance.

For RTP_SAT:

```text
RULE_FAMILY = R_SAT_v1
```

where:

```text
R_SAT_v1 = {S1,S2,S3,S4,S5,S6,S7}.
```

Because rule semantics are fixed by the encoding version and rule-family identifier, formula-specific executable code is not serialized.

## 3.7 BOUNDARIES

For the reduction:

```text
|B| = 0.
```

The section contains the canonical empty-boundary encoding.

## 3.8 ENDPOINTS

Stores:

```text
s
t
```

as vertex identifiers.

## 3.9 KLEIN

Stores the explicit Klein set.

For RTP_SAT:

```text
K = {k}.
```

## 3.10 INITIAL_STATE

RTP_SAT uses the canonical default:

```text
sigma_0(v) = OPEN
```

for every constructed vertex.

The section therefore encodes:

```text
DEFAULT OPEN
NO OVERRIDES.
```

## 3.11 TARGETS

Stores every reconciliation target explicitly.

For RTP_SAT:

```text
T_R = {c_1,...,c_m}.
```

---

# 4. Constructed graph counts

Let the source formula have:

```text
n variables
m clauses
```

with exactly three literals per clause.

The constructed vertex set contains:

```text
n+1   assignment-stage vertices
2n    Boolean-choice vertices
1     Klein vertex
m+1   clause-stage vertices
3m    literal vertices
1     terminal vertex
```

Therefore:

```text
|V_phi|
= (n+1) + 2n + 1 + (m+1) + 3m + 1
= 3n + 4m + 4.
```

The edge set contains:

```text
4n    variable-gadget edges
2     assignment/Klein/clause bridge edges
6m    clause-gadget edges
1     final edge to t
```

Therefore:

```text
|E_phi| = 4n + 6m + 3.
```

Both are linear in:

```text
n+m.
```

---

# 5. Successful trajectory length

Every successful trajectory has:

```text
2n
```

variable-gadget transitions,

then:

```text
a_n -> k
k -> c_0
```

giving two transitions,

then:

```text
2m
```

clause-gadget transitions,

and finally:

```text
c_m -> t.
```

Thus every successful trajectory has exactly:

```text
ell = 2n + 2m + 3
```

transitions.

---

# 6. Theorem PO-1: Canonical Encoding

**Theorem.**

For every RTP_SAT instance produced by the reduction:

```text
N >= 2n + 2m + 3.
```

### Proof

RTP-ENC-v1 contains one explicit nonempty record for every graph vertex.

Therefore:

```text
N >= |V_phi|.
```

By Section 4:

```text
|V_phi| = 3n + 4m + 4.
```

Subtract the successful trajectory length:

```text
(3n + 4m + 4)
-
(2n + 2m + 3)
=
n + 2m + 1.
```

Since:

```text
n >= 0
m >= 0,
```

we have:

```text
n + 2m + 1 > 0.
```

Hence:

```text
N
>=
3n + 4m + 4
>
2n + 2m + 3.
```

Therefore:

```text
N >= 2n + 2m + 3.
```

Thus:

```text
PO-1 Canonical Encoding = PASS.
```

QED.

---

# 7. Canonical certificate encoding

The exact certificate serialization is defined in:

```text
RTP_CERTIFICATE_CONSTANT_8.md
```

For instance size:

```text
N = |RTP-ENC-v1(I)|,
```

the canonical certificate uses:

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
BLOCK_N(omega_0) || ... || BLOCK_N(omega_(m-1)).
```

Here:

```text
|CERT_TAG| = 1
|FW_N(x)| = N
|BLOCK_N(x)| = 2N
```

for every bounded block `|x| <= N`.

The exact serialized length is:

```text
|Enc_I(C)| = 1 + (5m+2)N.
```

Since `m <= N`:

```text
|Enc_I(C)|
<=
5N^2 + 2N + 1
<=
8N^2
```

for every `N >= 1`.

Thus:

```text
c_RTP = 8
```

is a proved canonical language constant, not an asymptotic placeholder.

---

# 8. Constructed transition blocks

For RTP_SAT every transition uses one of seven fixed schemas.

Each accepted transition contains only a constant number of:

- vertex references;
- variable/clause/literal indices;
- rule identifiers;
- state effects;
- relational effects;
- witness references;
- provenance-parent references.

Each such reference requires:

```text
O(log N)
```

bits.

Therefore there exists a fixed encoding constant `d`, independent of `phi`, such that:

```text
|delta_i| <= d log(N+2)
|omega_i| <= d log(N+2)
```

for every transition produced by the reduction.

Consequently:

```text
|delta_i| = O(log N)
|omega_i| = O(log N).
```

For all sufficiently large `N`:

```text
d log(N+2) <= N.
```

The finitely many smaller source formulas may be mapped to precomputed constant-size canonical YES/NO RTP instances, as standard for finite exceptions in a polynomial-time many-one reduction.

Hence every produced certificate satisfies the RTP per-transition bound:

```text
|delta_i| <= N
|omega_i| <= N.
```

---

# 9. Trajectory encoding bound

By PO-1:

```text
ell <= N.
```

Also:

```text
|V| <= N
```

because every vertex has an explicit nonempty instance record.

Thus:

```text
w <= ceil(log_2 N)
```

for `N >= 2`.

The trajectory contains:

```text
ell + 1 <= N+1
```

vertex identifiers.

Therefore:

```text
|pi|
=
O(N log N).
```

In particular:

```text
|pi| = O(N^2).
```

---

# 10. Delta and witness sequence bounds

Because:

```text
ell <= N
```

and each block obeys:

```text
|delta_i| <= N
|omega_i| <= N,
```

we have:

```text
|Delta|
<=
N * N
=
N^2
```

and:

```text
|Omega|
<=
N * N
=
N^2.
```

For the actual RTP_SAT construction the stronger bound holds:

```text
|Delta| + |Omega|
=
O(N log N),
```

because every transition block has constant arity.

---

# 11. Fixed global certificate constant

The exact derivation in `RTP_CERTIFICATE_CONSTANT_8.md` gives:

```text
|Enc_I(C)| = 1 + (5m+2)N.
```

Using `m <= N`:

```text
|Enc_I(C)|
<=
5N^2 + 2N + 1.
```

For every `N >= 1`:

```text
8N^2 - (5N^2 + 2N + 1)
=
(3N+1)(N-1)
>=
0.
```

Therefore:

```text
|Enc_I(C)| <= 8N^2
```

for every canonical RTP instance, with no finite-exception argument.

Thus:

```text
c_RTP = 8
```

is formally established.

The stronger envelope:

```text
|Enc_I(C)| <= 5N^2 + 2N + 1
```

also holds, but `8N^2` remains the canonical language ceiling.

---

# 12. Theorem PO-11: Certificate Bounds

**Theorem.**

Every successful certificate produced by the 3SAT reduction satisfies:

```text
ell <= N
```

and:

```text
|C| = O(N^2).
```

### Proof

Section 5 gives:

```text
ell = 2n + 2m + 3.
```

PO-1 gives:

```text
N >= 2n + 2m + 3.
```

Therefore:

```text
ell <= N.
```

Sections 7-11 show:

```text
|C| <= 8N^2.
```

Hence:

```text
|C| = O(N^2).
```

Therefore:

```text
PO-11 Certificate Bounds = PASS.
```

QED.

---

# 13. Source 3SAT encoding

Let:

```text
L = |Enc_3SAT(phi)|
```

be the bit length of the input formula under any ordinary explicit 3-CNF encoding.

Normalize the source variable set so that:

```text
{x_1,...,x_n}
```

is exactly the set of variables appearing in `phi`.

For a nonempty 3-CNF formula:

```text
m <= O(L)
```

because every clause has an explicit representation, and:

```text
n <= 3m
```

because every normalized variable occurs in at least one of the `3m` literal positions.

Therefore:

```text
n+m = O(L).
```

The degenerate empty formula is handled by a fixed constant-size YES instance.

---

# 14. Output-size theorem

Let:

```text
q = n+m+2.
```

All graph identifiers and parameters are bounded by `O(q)`, hence each uses:

```text
O(log q)
```

bits.

There are:

```text
O(n+m)
```

vertex, edge, target, and metadata records.

Therefore:

```text
N
=
O((n+m) log(n+m+2)).
```

Since:

```text
n+m = O(L),
```

we obtain:

```text
N
=
O(L log(L+2)).
```

and therefore:

```text
N = poly(L).
```

---

# 15. Construction algorithm

Given explicit 3-CNF formula `phi`:

1. parse the formula;
2. normalize appearing variable symbols to dense indices `1,...,n`;
3. emit assignment-stage and Boolean-choice vertices;
4. emit the four edges for each variable gadget;
5. emit the Klein vertex and two bridge edges;
6. for each source clause:
   - emit its clause-stage vertex;
   - emit three literal vertices carrying polarity and variable index;
   - emit the six clause-gadget edges;
7. emit terminal `t` and edge `c_m -> t`;
8. emit `R_SAT_v1`, empty boundary set, default OPEN initial state, `K={k}`, and `T_R={c_1,...,c_m}`;
9. serialize the result using RTP-ENC-v1.

No stage performs search over Boolean assignments.

No stage invokes the RTP verifier.

The construction is syntactic.

---

# 16. Construction time

Parsing requires:

```text
O(L)
```

symbol visits.

Dense variable normalization can be implemented in polynomial time.

Even using a deliberately conservative deterministic comparison-based implementation, it requires at most:

```text
O(L^2)
```

bit operations.

Emission produces:

```text
O(L log(L+2))
```

output bits.

Therefore the total reduction time is bounded by:

```text
O(L^2)
```

under a conservative bit-cost model.

Hence:

```text
f(phi) = I_phi
```

is computable in polynomial time.

---

# 17. Theorem PO-12: Polynomial Construction

**Theorem.**

The mapping:

```text
f : 3SAT -> RTP_SAT
```

is computable in polynomial time and produces polynomial-size output.

### Proof

By Section 14:

```text
|f(phi)|
=
O(L log(L+2))
=
poly(L).
```

By Section 16:

```text
time_f(L)
=
O(L^2)
=
poly(L).
```

Therefore the mapping is a polynomial-time many-one reduction construction.

Thus:

```text
PO-12 Polynomial Construction = PASS.
```

QED.

---

# 18. Completed proof-obligation ledger

```text
PO-1  Canonical Encoding            PASS
PO-2  Assignment Memory             PASS
PO-3  Fixed Rule-Set Semantics      PASS
PO-4  Polynomial Primitive Checks   PASS
PO-5  Assignment Exclusivity        PASS
PO-6  Assignment Completeness       PASS
PO-7  Clause Soundness              PASS
PO-8  No Bypass Path                PASS
PO-9  Terminal Equivalence          PASS
PO-10 Provenance Monotonicity       PASS
PO-11 Certificate Bounds            PASS
PO-12 Polynomial Construction       PASS
```

All stated proof obligations for the candidate 3SAT reduction are discharged under RTP-ENC-v1.

---

# 19. NP-hardness theorem

The previously proved semantic directions give:

```text
phi in 3SAT
iff
f(phi) in RTP_SAT.
```

PO-12 establishes that `f` is polynomial-time computable.

Therefore:

```text
3SAT <=p RTP_SAT.
```

Because 3SAT is NP-hard:

```text
RTP_SAT is NP-hard.
```

Since:

```text
RTP_SAT subseteq RTP,
```

it follows that:

```text
RTP is NP-hard.
```

---

# 20. NP-completeness theorem

The canonical RTP specification already establishes:

```text
RTP in NP.
```

Section 19 establishes:

```text
RTP is NP-hard.
```

Therefore:

```text
RTP is NP-complete.
```

under the canonical bounded problem definition and RTP-ENC-v1 encoding.

Formally:

```text
RTP in NP
and
3SAT <=p RTP
=>
RTP is NP-complete.
```

QED.
