# Set Calculus Core

This directory will hold the canonical mathematical vocabulary and formal transform model.

## Candidate primitives

```text
Set
Member
Relationship
State
Transform
Resolution
Provenance
```

These names remain candidates until formally defined.

## Initial invariants

### Transform-produced properties

A property produced by a transform cannot automatically be projected backward onto its unresolved input.

```text
property(T(x)) does not imply property(x)
```

unless an explicit rule establishes that implication.

### Unresolved is not unknown

An unresolved state may contain constraints, relationships, provenance, and possible resolution paths even when it does not contain a unique final value.

```text
Unresolved != Unknown
```

### Provenance survives transformation

A resolution should retain enough trace to determine what admitted inputs, relationships, and transforms produced it.

### Explicit failure

The system should represent failure to resolve rather than silently inventing a value.

Candidate terminal or reportable states include:

- unresolved
- resolved
- reversible resolution
- closure
- logical failure

The exact vocabulary remains subject to formalization.

## First formalization task

Define each primitive independently of programming-language implementation, then express a minimal canonical example in both mathematical notation and machine-readable form.

## Active formalizations

- `TRAJECTORY_ADMISSIBILITY_AND_RESOLUTION_DEPTH.md` - trajectory admissibility, composition, boundary compatibility, representation structures, depth-indexed requirements, requirement entailment, and supported resolution depth.
- `ANCHOR_FOUNDATIONAL_SPACE.md` - minimal foundational resolution-space substrate, primitive objects, notation, reconciliation states, closure, provenance, and Anchor axioms.
- `RESOLUTION_TRANSITION_PROBLEM.md` - finite decision problem over an Anchor projection; canonical linear certificate bound and polynomial verifier establish bounded RTP membership in NP.
- `RTP_NP_HARDNESS_REDUCTION.md` - candidate polynomial-time many-one reduction from 3SAT to RTP, including construction, completeness/soundness argument, and explicit proof obligations.
- `RTP_ACTIVE_RELATIONAL_LEDGER.md` - canonical finite representation of active relational state; exact persistent Boolean-assignment representation; discharges PO-2.
- `RTP_SAT_ASSIGNMENT_INVARIANTS.md` - proves assignment exclusivity and assignment completeness for the 3SAT reduction; discharges PO-5 and PO-6.
- `RTP_SAT_CLAUSE_SOUNDNESS.md` - proves clause soundness, global no-bypass topology, and terminal reconciliation equivalence; discharges PO-7, PO-8, and PO-9.
- `RTP_SAT_RULE_COMPLEXITY.md` - proves `R_SAT` is a fixed seven-schema rule family and every primitive rule check is polynomial-time; discharges PO-3 and PO-4.
- `RTP_SAT_PROVENANCE_MONOTONICITY.md` - proves append-only verifier-derived provenance, assertion lineage retention, and anti-forgery/anti-erasure invariants; discharges PO-10.
- `RTP_CANONICAL_ENCODING_AND_SIZE_PROOFS.md` - freezes `RTP-ENC-v1`, proves the linear trajectory/certificate bounds and polynomial construction, discharges PO-1/PO-11/PO-12, and completes the NP-hardness/NP-completeness theorem.
- `artifacts/Resolution_Space_Framework.pdf` - preserved primary source artifact for the Anchor/RTP line; see `../provenance/PHOTONIC_MODEL_ANCHOR_LINEAGE.md` for project provenance.

- `RTP_CANONICAL_VERIFIER_REGISTRIES.md` - freezes `RTP-RULES-v1` and `RTP-BOUNDARIES-v1`, canonical predicates, version immutability, and a common `O(W^2)` primitive runtime bound.
- `RTP_VERIFIER_STATE_SIZE.md` - proves every canonical verifier state has size `O(N^2)` and derives a uniform conservative `O(N^5)` verifier bound for general RTP.

- `RTP_CERTIFICATE_CONSTANT_8.md` - gives the exact canonical certificate serialization and proves `|C| <= 8N^2` for every canonical instance, establishing `c_RTP = 8` without hidden asymptotic constants.
