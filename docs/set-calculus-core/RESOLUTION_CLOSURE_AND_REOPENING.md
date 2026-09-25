# Resolution Closure and Reopening

**Status:** Canonical Set Calculus formalization candidate  
**Scope:** Resolution stopping, closure receipts, reopening, reclosure, protected residual conservation  
**Applies to:** Set Calculus Core  
**Source references:**
- `SRC-HYPERTRIANGLE-V1P0P2-20260925`
- `SRC-TENET-PASS022-20260925`

---

# 1. Purpose

Set Calculus requires an explicit account of four related questions:

1. When has a resolver received enough evidence to stop?
2. What exactly is preserved when a result reaches closure?
3. What happens when new admissible evidence arrives after closure?
4. Can a result count as resolved when its apparent success was obtained by worsening another protected residual?

This document formalizes two rules:

```text
First-Terminal-Prefix Stopping
Protected Residual Conservation
```

The first governs when resolution computation may stop and how a stopped result may later reopen.

The second governs whether a proposed closure is admissible when other protected residuals exist.

Together they establish:

```text
closure may be locally terminal without being permanently immutable

provenance may grow monotonically while resolution status changes

target closure does not justify protected residual sacrifice
```

---

# 2. Core distinctions

The following distinctions are normative for this document.

```text
Identity != State

PARTIAL != UNRESOLVED

TerminalClosure != PermanentClosure

TargetClosed != Resolved

ProvenanceMonotonicity != ResolutionMonotonicity

Superseded != Erased
```

A resolution is evaluated against a declared target, evidence set, rule set, and closure contract.

A later change in evidence may therefore change the current resolution without rewriting the historical fact that an earlier resolution was valid under its earlier evidence horizon.

---

# 3. Resolver configuration

Let a Set Calculus resolution configuration be

[
q_n =
langle
x,
E_n,
Theta_x,
Omega,
S_n,
R_n,
P_n
angle
]

where:

- (x) is the target object;
- (E_n) is the admitted evidence prefix through evidence item (n);
- (Theta_x) is the declared terminal contract for (x);
- (Omega) is the active rule set and rule version;
- (S_n) is the current resolved state structure;
- (R_n) is the residual structure;
- (P_n) is the accumulated provenance.

Let the ordered admitted evidence stream be

[
E =
langle
e_1,e_2,ldots
angle .
]

Its prefix through (n) is

[
E_n =
langle
e_1,ldots,e_n
angle .
]

Evidence order is provenance-bearing and must not be silently rewritten.

---

# 4. Terminal contract

Let the declared terminal contract for target (x) be

[
Theta_x =
{	heta_1,	heta_2,ldots,	heta_m}.
]

Each (	heta_i) is a required resolution predicate.

For the purposes of terminal evaluation, each predicate resolves to:

```text
TRUE
FALSE
UNRESOLVED
```

Define:

[
Terminal_{Theta_x}(q_n)
iff
igwedge_{i=1}^{m}
[	heta_i(q_n)=TRUE].
]

Therefore:

[
UNRESOLVED 
eq TRUE
]

and

[
PARTIAL 
eq TERMINAL.
]

Terminality requires positive satisfaction of every required terminal predicate.

Absence of failure is not sufficient.

---

# 5. Resolution epoch

A **resolution epoch** is a contiguous period during which one active resolution attempt is evaluated against a fixed target identity and declared terminal contract.

Let epoch (k) be:

[
mathcal{E}_k.
]

Each epoch has:

- a start evidence index;
- a terminal contract;
- a rule/version set;
- an evidence horizon;
- zero or one active closure receipt.

A reopening ends the current closed epoch and begins a new active resolution epoch.

---

# 6. First-Terminal-Prefix stopping

The source construction motivating this rule defines the earliest evidence prefix satisfying all declared terminal requirements and stops at that prefix because later receipts need not preserve terminality.

Set Calculus generalizes that construction to arbitrary declared resolution contracts.

---

## SC-FTP-1: First-Terminal-Prefix Rule

If at least one evidence prefix satisfies the terminal contract, define:

[
n_k^*
=
min
{
n
mid
n ge start_k
land
Terminal_{Theta_x}(q_n)
}.
]

Then (n_k^*) is the **first terminal prefix** of epoch (k).

The resolver MUST stop active resolution for that epoch at (n_k^*).

Formally:

[
Terminal_{Theta_x}(q_{n_k^*})
Rightarrow
STOP(mathcal{E}_k).
]

Stopping means that the declared resolution obligation has been satisfied for the current evidence horizon.

It does not assert permanence under future evidence.

---

## SC-FTP-2: First-Terminal Minimality

For every evidence prefix before the first terminal prefix:

[
orall j,
quad
start_k le j < n_k^*
Rightarrow

eg Terminal_{Theta_x}(q_j).
]

Therefore (n_k^*) is minimal.

No later terminal prefix may be substituted for (n_k^*) as the original stopping point.

The original stopping horizon remains part of provenance.

---

## SC-FTP-3: Closure Receipt

At (n_k^*), the resolver emits a closure receipt:

[
C_k =
langle
id_x,
k,
n_k^*,
Theta_x,
Omega_k,
S_{n_k^*},
R_{n_k^*},
P_{n_k^*},
W_k
angle
]

where:

- (id_x) is the identity of the resolved target;
- (k) is the resolution epoch;
- (n_k^*) is the first terminal evidence horizon;
- (Theta_x) is the terminal contract;
- (Omega_k) is the active rule/version set;
- (S_{n_k^*}) is the resolved state;
- (R_{n_k^*}) is the residual structure at closure;
- (P_{n_k^*}) is provenance through closure;
- (W_k) is the terminal closure witness.

The receipt asserts:

[
Resolved
(
x
mid
E_{n_k^*},
Theta_x,
Omega_k
).
]

It does not assert:

[
Resolved
(
x
mid
E_{infty}
).
]

Closure is evidence-horizon specific.

---

## SC-FTP-4: Closure Receipt Persistence

Once produced, a closure receipt remains in provenance.

For all later configurations (q_r), where (r > n_k^*):

[
C_k in P_r.
]

A later resolution may supersede (C_k) as the current result.

It may not erase the historical closure event.

Therefore:

```text
supersession != deletion
```

and:

```text
reopening != retroactive invalidation of provenance
```

---

## SC-FTP-5: Resolution Non-Monotonicity

Terminal resolution is not generally monotone under arbitrary evidence extension.

Therefore:

[
Terminal(q_n)

otRightarrow
Terminal(q_{n+1}).
]

A newly admitted item may introduce:

- conflicting evidence;
- additional residual;
- changed uncertainty;
- failed authority;
- failed invariant;
- failed provenance;
- failed protected-residual condition;
- a newly material relation.

Thus Set Calculus does not assume:

[
Resolved(E_n)
Rightarrow
Resolved(E_{n+1}).
]

---

## SC-FTP-6: Evidence-Driven Reopening

Let (C_k) be the current active closure receipt.

Let new evidence (e_r), with (r > n_k^*), become available.

Reopening requires all of the following:

### Premise R1: Admissibility

[
Admissible(e_r,Omega).
]

### Premise R2: Material relevance

[
Relevant(e_r,x,Theta_x).
]

### Premise R3: Terminal failure after incorporation

Let

[
q_r =
Resolve(x,E_r,Theta_x,Omega).
]

Then:

[

eg Terminal_{Theta_x}(q_r).
]

If all three premises hold:

[
Reopen(C_k,e_r).
]

Formally:

[
rac{
Admissible(e_r,Omega)
qquad
Relevant(e_r,x,Theta_x)
qquad

eg Terminal_{Theta_x}(q_r)
}{
Reopen(C_k,e_r)
}.
]

Inadmissible evidence does not reopen a resolved object.

Irrelevant evidence does not reopen a resolved object.

---

## SC-FTP-7: No Retroactive Erasure

If evidence (e_r) reopens closure (C_k), the following statements may both be true:

[
Resolved
(
x
mid
E_{n_k^*}
)
]

and

[

eg Resolved
(
x
mid
E_r
).
]

There is no contradiction because the evidence horizons differ.

The historical statement:

```text
Ck was the valid closure under evidence horizon n*k
```

remains preserved.

The current statement becomes:

```text
Ck is superseded as the active closure
```

Thus:

[
HistoricalValidity(C_k)
land
Superseded(C_k)
]

is admissible.

---

## SC-FTP-8: Reclosure Creates a New Epoch

A reopening begins a new resolution epoch:

[
mathcal{E}_{k+1}.
]

Let its starting evidence index be (r).

The resolver seeks:

[
n_{k+1}^*
=
min
{
n
mid
nge r
land
Terminal_{Theta_x}(q_n)
}.
]

If such a prefix exists, a new closure receipt is emitted:

[
C_{k+1}.
]

The resolver MUST NOT rewrite (C_k) into (C_{k+1}).

Instead:

[
C_k
prec_P
C_{k+1}.
]

The provenance history is:

```text
ACTIVE EPOCH k
    |
    v
FIRST TERMINAL PREFIX
    |
    v
CLOSURE Ck
    |
    | new admissible material evidence
    v
REOPEN
    |
    v
ACTIVE EPOCH k+1
    |
    v
FIRST TERMINAL PREFIX
    |
    v
CLOSURE Ck+1
```

Both closure receipts remain reconstructible.

Only the newest non-superseded receipt is active.

---

# 7. Active closure

Define:

[
Active(C_k)
]

iff (C_k) has been emitted and no later valid reopening event has superseded it.

Formally:

[
Active(C_k)
iff

eg
exists r>n_k^*:
Reopen(C_k,e_r).
]

The current closed state of (x) is governed by the most recent active closure receipt.

---

# 8. Provenance monotonicity

Set Calculus permits resolution to reopen while requiring provenance to remain monotone.

For successive configurations:

[
P_n
preceq_P
P_{n+1}.
]

Therefore:

[
ProvenanceMonotone
]

does not imply:

[
ResolutionMonotone.
]

The canonical distinction is:

```text
provenance grows

resolution may close
resolution may reopen
resolution may close again
```

---

# 9. Public state after reopening

Reopening removes the active terminal classification.

The ordinary Set Calculus resolution algebra then determines the new public state.

The result may be:

```text
PARTIAL
UNRESOLVED
INVALID
```

according to the active evidence and rules.

Reopening itself does not force any one of these outcomes.

In particular:

```text
REOPEN != INVALID
```

and:

```text
REOPEN != UNRESOLVED
```

The resolution algebra determines the resulting classification.

---

# 10. Protected residuals

A resolver may operate over more than one residual.

Let the residual family be:

[
R =
{r_1,r_2,ldots,r_n}.
]

Let:

[
P_R subseteq R
]

be the declared protected residual family.

A protected residual is one whose worsening constrains admissible resolution.

For each protected residual (pin P_R), let:

[
preceq_p
]

be the declared typed admissibility order.

Then:

[
p^{after}
preceq_p
p^{before}
]

means that the protected residual has not worsened.

Residual orders are typed.

Set Calculus does not require heterogeneous residuals to be converted into one scalar quantity.

---

# 11. Authorized residual exchange

A protected residual may worsen only under an explicitly declared exchange law.

Define:

[
AuthorizedExchange(T,p,Omega)
]

iff the active rule set contains an applicable rule authorizing the change to protected residual (p).

An authorized exchange rule MUST identify at least:

1. the affected residual;
2. the transform or operation;
3. the allowed direction or bound of change;
4. the authority permitting the exchange;
5. the relevant scope/context;
6. the provenance record required.

If no applicable exchange rule exists:

[

eg AuthorizedExchange(T,p,Omega).
]

Set Calculus therefore applies:

```text
silence != authorization
```

---

# 12. Protected Residual Conservation

The source construction motivating this rule states that closure cannot be obtained by hiding or worsening a protected residual unless an explicitly typed exchange law permits the change.

Set Calculus generalizes that rule to resolution transforms.

---

## SC-PRC-1: Protected Residual Conservation

Let:

[
T:qightarrow q'
]

be a proposed resolution-producing transform.

Protected Residual Conservation requires the following premises.

### Premise P1: Declared protected family

[
P_Rsubseteq R.
]

The protected residual family is declared before closure evaluation.

### Premise P2: Typed comparison

For every:

[
pin P_R,
]

a valid typed order:

[
preceq_p
]

is available.

### Premise P3: Post-transform evaluability

For every protected residual:

[
p^{after}
]

must be known or validly resolved.

An unresolved protected residual is not silently interpreted as conserved.

### Premise P4: Non-sacrifice or authorized exchange

For every:

[
pin P_R,
]

either:

[
p^{after}preceq_p p^{before}
]

or:

[
AuthorizedExchange(T,p,Omega).
]

### Premise P5: Provenance

If an authorized exchange occurs:

[
ExchangeRecord(T,p)in Prov(q').
]

### Conclusion

If P1-P5 hold:

[
PRC(T,q,q').
]

Inference form:

[
rac{
P_Rsubseteq R
qquad
orall pin P_R:
left(
p^{after}preceq_p p^{before}
lor
AuthorizedExchange(T,p,Omega)
ight)
qquad
ProvPreserved
}{
PRC(T,q,q')
}.
]

For any transform that claims resolution:

[
Resolve(T,q,q')
Rightarrow
PRC(T,q,q').
]

---

## SC-PRC-2: No Hidden Residual Transfer

Let (r_t) be the target residual.

Suppose:

[
r_t^{after}
<
r_t^{before}.
]

Suppose further that for some protected residual (p):

[
p^{after}
succ_p
p^{before}
]

and:

[

eg AuthorizedExchange(T,p,Omega).
]

Then:

[

eg PRC(T,q,q').
]

Therefore:

[

eg AdmissibleClosure(T,q').
]

This remains true even when:

[
r_t^{after}=0.
]

Thus:

[
TargetClosed

otRightarrow
Resolved.
]

A target residual cannot be declared globally resolved merely because unresolved cost has been transferred elsewhere.

---

## SC-PRC-3: Protected Vector Closure

Where protected residuals exist, closure is evaluated over the declared protected residual structure rather than the target residual alone.

Let:

[
ec{R}(q)
=
langle
r_t,
p_1,
p_2,ldots,p_k
angle .
]

Then:

[
r_t=0
]

is insufficient for admissible resolution.

Resolution requires:

[
TargetClosed(q')
land
PRC(T,q,q').
]

Therefore:

[
TargetClosed(q')
land

eg PRC(T,q,q')
Rightarrow

eg Resolved(q').
]

This establishes:

```text
target closure
+
protected residual conservation
=
candidate admissible closure
```

subject to all other Set Calculus terminal requirements.

---

# 13. Counterexample: Compensating Closure

Consider a resolver with two residuals:

[
R=
{r_A,r_B}.
]

Let:

[
P_R=
{r_B}.
]

Assume lower values are preferred under the declared order.

Initial state:

[
r_A^{before}=10
]

and:

[
r_B^{before}=2.
]

A transform (T) produces:

[
r_A^{after}=0
]

and:

[
r_B^{after}=8.
]

The target residual closes:

[
r_A^{after}=0.
]

But the protected residual worsens:

[
8succ 2.
]

Assume:

[

eg AuthorizedExchange(T,r_B,Omega).
]

Therefore:

[

eg PRC(T,q,q').
]

By SC-PRC-2:

[

eg AdmissibleClosure(T,q').
]

By SC-PRC-3:

[

eg Resolved(q').
]

The correct resolution record is:

```text
TARGET RESIDUAL
CLOSED

PROTECTED RESIDUAL
WORSENED

AUTHORIZED EXCHANGE
NO

PROTECTED RESIDUAL CONSERVATION
FAIL

FINAL RESOLUTION
INVALID
```

The transform achieved apparent target success by transferring unresolved cost into a protected component.

Set Calculus rejects the closure.

---

# 14. Interaction between FTP and PRC

Protected Residual Conservation may be included as a terminal predicate.

For example:

[
Theta_x =
{
	heta_{identity},
	heta_{state},
	heta_{authority},
	heta_{invariant},
	heta_{provenance},
	heta_{target},
	heta_{PRC}
}.
]

Then:

[
	heta_{PRC}(q_n)=TRUE
iff
PRC(q_{n-1},q_n).
]

A first terminal prefix therefore cannot occur while protected residual conservation fails.

If:

[
TargetClosed(q_n)=TRUE
]

but:

[
PRC(q_{n-1},q_n)=FALSE,
]

then:

[
Terminal(q_n)=FALSE.
]

Thus:

```text
target success cannot create terminal closure
while protected sacrifice remains unresolved
```

---

# 15. Reopening from protected residual evidence

Protected residual evidence may also trigger reopening.

Suppose closure receipt (C_k) was valid under:

[
E_{n_k^*}.
]

Later admissible evidence (e_r) establishes that a protected residual was worsened by the closure transform.

Then:

[
Admissible(e_r)
]

and:

[
Relevant(e_r,	heta_{PRC})
]

and:

[
	heta_{PRC}(q_r)=FALSE.
]

Therefore by SC-FTP-6:

[
Reopen(C_k,e_r).
]

The earlier closure receipt remains in provenance.

Its active status is superseded.

The new resolution epoch must determine whether the result becomes:

```text
PARTIAL
UNRESOLVED
INVALID
```

or can later reach a new admissible closure.

---

# 16. Canonical state machine

```text
ACTIVE RESOLUTION
        |
        | all terminal predicates TRUE
        v
FIRST TERMINAL PREFIX
        |
        v
CLOSURE RECEIPT Ck
        |
        v
STOPPED / ACTIVE CLOSURE
        |
        | new admissible material evidence
        v
RE-EVALUATE TERMINAL CONTRACT
        |
        +-----------------------------+
        |                             |
        | predicates remain TRUE      | predicate becomes non-TRUE
        v                             v
CLOSURE REMAINS ACTIVE             REOPEN
                                      |
                                      v
                               NEW RESOLUTION EPOCH
                                      |
                                      | terminal predicates TRUE
                                      v
                               NEW CLOSURE RECEIPT
```

At every transition:

```text
provenance is retained
```

and:

```text
historical closure receipts are not rewritten
```

---

# 17. Core invariants

## 17.1 Stopping invariant

[
FirstTerminalPrefix
Rightarrow
StopCurrentEpoch.
]

## 17.2 Evidence-horizon invariant

[
Closure
=
ClosureRelativeToDeclaredEvidenceHorizon.
]

## 17.3 Reopening invariant

[
NewMaterialAdmissibleEvidence
land
TerminalFailure
Rightarrow
Reopen.
]

## 17.4 Provenance invariant

[
P_n
preceq_P
P_{n+1}.
]

## 17.5 Historical closure invariant

[
Superseded(C_k)

otRightarrow
Erased(C_k).
]

## 17.6 Protected residual invariant

[
Resolved(T)
Rightarrow
PRC(T).
]

## 17.7 No-sacrifice invariant

[
TargetClosed(T)
land
ProtectedSacrifice(T)
land

eg AuthorizedExchange(T)
Rightarrow

eg Resolved(T).
]

---

# 18. Consequences for Set Calculus

These rules establish several consequences for the Core model.

### 18.1 Resolution is evidence-relative

A closure statement identifies the evidence horizon and rules under which it was obtained.

### 18.2 Resolution may reopen

Closure does not imply permanent immunity from later admissible evidence.

### 18.3 Provenance remains monotone

Reopening and reclassification extend the trace instead of rewriting it.

### 18.4 Reclosure is a new event

A reopened target that later resolves receives a new closure receipt.

### 18.5 Resolution is relational

A target cannot be evaluated independently of residuals explicitly declared relevant to its admissibility.

### 18.6 Protection is typed

Protected residuals retain their own orders and domains.

They do not require arbitrary scalar collapse.

### 18.7 Authorization is explicit

A protected sacrifice requires an applicable typed exchange rule.

Implicit permission is insufficient.

---

# 19. Source anchors

## SRC-HYPERTRIANGLE-V1P0P2-20260925

**Repository source:**

`docs/set-calculus-core/artifacts/HYPERTRIANGLE_FINAL_EN (1).pdf`

### First-terminal-prefix source

```text
Source: SRC-HYPERTRIANGLE-V1P0P2-20260925
Anchor: §11.3 First-terminal-prefix stopping, Equation (41), PDF p. 16
```

Source structure:

[
n^*
=
min
{
n:P_n=true
}.
]

The source further states that terminality need not be monotone under arbitrary receipt appending and prescribes stopping at the first terminal prefix or screening later candidate receipts.

Set Calculus generalizes this from the HyperTriangle certification protocol to declared Set Calculus terminal contracts.

### Scoped closure source

```text
Source: SRC-HYPERTRIANGLE-V1P0P2-20260925
Anchor: Theorem 12.1, Scoped closure of the HyperTriangle architecture, PDF pp. 16-17
```

The source distinguishes internal closure of the declared certification architecture from application-specific inputs.

Set Calculus uses the same structural distinction when separating resolver completeness, evidence completeness, and instance resolution.

---

## SRC-TENET-PASS022-20260925

**Repository source:**

`docs/set-calculus-core/artifacts/TENET_CAUSAL_CUMULATIVE_PASS022_EN_20260925.pdf`

### Protected non-sacrifice source

```text
Source: SRC-TENET-PASS022-20260925
Anchor: §213.5 K5 - Protected non-sacrifice, PDF p. 97
```

The source requires:

[
R^{after}_{prot}
preceq
R^{before}_{prot}
]

unless a separately typed exchange law explicitly authorizes the change.

Set Calculus generalizes this rule into SC-PRC-1 through SC-PRC-3.

### Countermodel source

```text
Source: SRC-TENET-PASS022-20260925
Anchor: §214 PASS019 executable ablation, Compensating Sator countermodel, PDF pp. 97-98
```

The source identifies the failure mode:

```text
raw success paid by protected sacrifice
```

The numerical counterexample in this document is a Set Calculus construction illustrating that rule. It is not copied from the source.

### Provenance persistence source

```text
Source: SRC-TENET-PASS022-20260925
Anchor: §213.3 K3 - Trace and provenance, PDF p. 97
```

The source requires retained lineage and append-only trace behavior through reciprocal closure.

Set Calculus combines that provenance discipline with reopening so that historical closure receipts remain preserved after supersession.

---

# 20. Source/reference distinction

The source documents motivate specific structures formalized here.

The following are source-derived structures:

```text
first-terminal-prefix stopping
non-monotone terminality under later receipts
protected non-sacrifice
typed authorized exchange
append-only provenance/lineage
```

The following are Set Calculus generalizations introduced by this document:

```text
resolution epochs
closure receipt Ck
Active(Ck)
SC-FTP-1 through SC-FTP-8
SC-PRC-1 through SC-PRC-3
integration with VALID/PARTIAL/UNRESOLVED/INVALID
reopening transition semantics
the numerical compensating-closure counterexample
```

Source provenance is retained without treating the Set Calculus generalization as a verbatim theorem of either source document.

---

# 21. Summary

The canonical stopping rule is:

[
oxed{
n_k^*
=
min
{
n
mid
Terminal_{Theta_x}(q_n)
}
}
]

The canonical reopening rule is:

[
oxed{
Admissible(e_r)
land
Relevant(e_r)
land

eg Terminal(q_r)
Rightarrow
Reopen
}
]

The canonical provenance rule is:

[
oxed{
Provenance grows
quad
while
quad
Resolution may close, reopen, and close again
}
]

The canonical protected residual rule is:

[
oxed{
Resolved(T)
Rightarrow
orall pin P_R:
left(
p^{after}preceq_p p^{before}
lor
AuthorizedExchange(T,p)
ight)
}
]

Therefore:

[
oxed{
TargetClosed
land
ProtectedSacrifice
land

eg AuthorizedExchange
Rightarrow

eg Resolved
}
]

These rules jointly provide Set Calculus with deterministic stopping, non-destructive reopening, and protected closure semantics.
