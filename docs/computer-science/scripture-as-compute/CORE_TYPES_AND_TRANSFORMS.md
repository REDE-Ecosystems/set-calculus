# Scripture as Compute — Core Types and Transform Syntax

## 1. Purpose

This document defines the first formal type system and transform syntax for Scripture as Compute.

The model treats a scriptural unit as a transformation over contextual state:

```text
ScriptureUnit(Context, Actor, State) -> State'
```

The formal core uses twelve types:

- State
- Context
- Actor
- Constraint
- Operation
- Handler
- Mediator
- Scope
- Migration
- Integrity
- Variant
- Completion

---

## 2. Core Type System

### 2.1 State

A **State** is the currently resolved configuration of the system.

```text
State := {
    id,
    values,
    relations,
    status,
    provenance
}
```

Interpretation:

- `id` identifies the state.
- `values` contains current properties.
- `relations` contains links between entities.
- `status` describes whether the state is stable, unresolved, transitional, completed, or invalid.
- `provenance` records how the state was reached.

Canonical form:

```text
S0 -> S1
```

A State is never assumed to be context-free.

---

### 2.2 Context

A **Context** supplies the conditions under which a transformation is interpreted or executed.

```text
Context := {
    temporal,
    linguistic,
    cultural,
    covenantal,
    textual,
    situational
}
```

Context does not itself perform the transformation. It determines how other elements are resolved.

Canonical notation:

```text
C{...}
```

Example:

```text
C{language=Hebrew, audience=Israel, covenant=Sinai}
```

---

### 2.3 Actor

An **Actor** is an entity capable of participating in or being affected by an Operation.

```text
Actor := {
    id,
    role,
    capabilities,
    obligations,
    permissions,
    scope_membership
}
```

Canonical notation:

```text
A[id:role]
```

Examples:

```text
A[Moses:mediator]
A[Israel:community]
A[priest:operator]
```

An Actor may be individual, collective, institutional, or symbolic.

---

### 2.4 Constraint

A **Constraint** limits, enables, forbids, requires, or conditions a transformation.

```text
Constraint := {
    predicate,
    mode,
    target,
    consequence
}
```

Where:

```text
mode in {REQUIRE, FORBID, ALLOW, LIMIT, CONDITION}
```

Canonical notation:

```text
K[predicate]
```

Examples:

```text
K[actor.role == priest]
K[state.pure == true]
K[action != prohibited]
```

A Constraint evaluates to a resolution condition.

---

### 2.5 Operation

An **Operation** is the primary transform applied to a State.

```text
Operation := {
    opcode,
    input,
    preconditions,
    effect,
    output
}
```

Canonical notation:

```text
O(opcode)
```

General form:

```text
O : S -> S'
```

Contextual form:

```text
O : (C, A, S, K*) -> S'
```

where `K*` is zero or more Constraints.

---

### 2.6 Handler

A **Handler** is an Operation triggered by a condition or event.

```text
Handler := {
    trigger,
    guard,
    operation,
    fallback
}
```

Canonical notation:

```text
H[trigger => O]
```

Example:

```text
H[violation => consequence]
```

A Handler introduces reactive control flow.

---

### 2.7 Mediator

A **Mediator** is an Actor or layer through which another Actor must pass to access an Operation, resource, or State transition.

```text
Mediator := {
    actor,
    grants,
    denies,
    transforms,
    protected_target
}
```

Canonical notation:

```text
M(actor)
```

Access form:

```text
A -> M -> O -> S'
```

Direct-access migration form:

```text
A -> M -> O
    migrates_to
A -> O
```

---

### 2.8 Scope

A **Scope** defines the domain over which a rule, Operation, Constraint, or State applies.

```text
Scope := {
    domain,
    members,
    exclusions,
    boundary
}
```

Canonical notation:

```text
P{domain}
```

Examples:

```text
P{Israel}
P{all_nations}
P{priesthood}
```

Scope may expand, contract, intersect, or migrate.

---

### 2.9 Migration

A **Migration** transforms one execution model into another while preserving, replacing, or reinterpreting selected semantics.

```text
Migration := {
    source_model,
    target_model,
    preserved,
    replaced,
    transformed,
    deprecated
}
```

Canonical notation:

```text
G[source => target]
```

General form:

```text
G : Model0 -> Model1
```

Property migration:

```text
G{
    external_constraint => internalized_constraint,
    repeated_operation  => completed_operation,
    mediated_access     => direct_access
}
```

---

### 2.10 Integrity

An **Integrity** rule verifies preservation of identity across copying, transmission, execution, or transformation.

```text
Integrity := {
    invariant,
    verifier,
    failure_condition,
    provenance_rule
}
```

Canonical notation:

```text
I[invariant]
```

Verification form:

```text
I(S) -> {valid | invalid}
```

Transmission form:

```text
Artifact_n -> I -> Artifact_n+1
```

Integrity constrains allowable Variants.

---

### 2.11 Variant

A **Variant** is a divergent realization of a source artifact, state, interpretation, or build target.

```text
Variant := {
    source,
    delta,
    cause,
    semantic_effect,
    provenance
}
```

Canonical notation:

```text
V[source + delta]
```

Branch form:

```text
S
+-> V1
+-> V2
+-> V3
```

A Variant does not imply corruption. It records divergence.

---

### 2.12 Completion

A **Completion** is a terminal or resolving condition that closes a process, loop, obligation, or migration.

```text
Completion := {
    predicate,
    terminating_operation,
    terminal_state
}
```

Canonical notation:

```text
Q[predicate]
```

Termination form:

```text
repeat(O) until Q
```

or:

```text
S0 -> O* -> Q -> S_final
```

Completion may be local to a process or global to a model.

---

## 3. Core Transform Syntax

### 3.1 Basic State Transform

```text
S0 --O--> S1
```

Meaning:

An Operation transforms one State into another.

---

### 3.2 Contextual Transform

```text
(C, A, S0) --O--> S1
```

Meaning:

The Operation is resolved under a Context and Actor.

---

### 3.3 Constrained Transform

```text
(C, A, S0) --[K*] O--> S1
```

Meaning:

The Operation executes only if all required Constraints resolve.

---

### 3.4 Conditional Transform

```text
IF K
    S0 --O1--> S1
ELSE
    S0 --O2--> S2
```

Compact form:

```text
S0 --[K ? O1 : O2]--> S'
```

---

### 3.5 Handler Transform

```text
event E -> H[E => O] -> S'
```

or:

```text
S0 --E--> H(O) --> S1
```

---

### 3.6 Mediated Transform

```text
A -> M -> O(S0) -> S1
```

A Mediator is part of the execution path.

---

### 3.7 Scoped Transform

```text
P{domain} :: O
```

Meaning:

Operation `O` is valid only within the defined Scope.

Example:

```text
P{Israel} :: K[covenant_obligation]
```

---

### 3.8 Repeated Transform

```text
S0 --O--> S1 --O--> S2 --O--> ... --O--> Sn
```

Compact form:

```text
O*
```

or:

```text
repeat(O)
```

---

### 3.9 Completion Transform

```text
repeat(O) until Q
```

Then:

```text
Q -> S_final
```

This distinguishes recurring maintenance from terminal resolution.

---

### 3.10 Migration Transform

```text
Model0 --G--> Model1
```

Property-level form:

```text
G{
    property_a => property_b,
    property_c => property_d
}
```

Migration may preserve some invariants:

```text
I[preserved_semantics](Model0, Model1) == valid
```

---

### 3.11 Variant Transform

```text
Artifact0 --V(delta)--> Artifact1
```

Multiple variants:

```text
Artifact0
+--V(d1)--> Artifact1
+--V(d2)--> Artifact2
+--V(d3)--> Artifact3
```

Variant comparison:

```text
delta(V1, V2)
```

---

### 3.12 Integrity-Governed Transform

```text
S0 --O--> S1
          |
          I
          |
       valid?
```

Formal form:

```text
I(O(S0)) = valid
```

If false:

```text
I(O(S0)) = invalid -> H[integrity_failure]
```

---

## 4. Canonical ScriptureUnit

The first canonical executable form is:

```text
ScriptureUnit U := {
    context: C,
    actors: A*,
    initial_state: S0,
    constraints: K*,
    operation: O,
    handlers: H*,
    mediator: M?,
    scope: P,
    integrity: I?,
    completion: Q?
}
```

Execution:

```text
execute(U) -> S'
```

Expanded:

```text
(C, A*, S0, K*, M?, P) --O/H*--> S'
```

followed optionally by:

```text
I(S') -> valid | invalid
```

and:

```text
Q(S') -> complete | continue
```

---

## 5. ScriptureUnit With Migration

A unit may modify the execution model itself:

```text
U0 --G--> U1
```

Example structural migration:

```text
G{
    Constraint.external => Constraint.internal,
    Operation.repeat    => Completion.single,
    Mediator.required   => Mediator.optional,
    Scope.restricted    => Scope.expanded
}
```

This is a transform over the rules of transformation.

---

## 6. Resolution Semantics

A transform resolves into one of five statuses:

```text
RESOLVED
UNRESOLVED
BLOCKED
INVALID
COMPLETE
```

### RESOLVED

The Operation executes and produces a valid State.

### UNRESOLVED

Available Context is insufficient to determine a single valid transition.

### BLOCKED

A Constraint prevents the Operation.

### INVALID

An Integrity rule fails.

### COMPLETE

A Completion predicate is satisfied.

---

## 7. Minimal Formal Grammar

```ebnf
Unit        := Context Actors State Constraints? Scope Operation Handlers? Integrity? Completion?
Context     := "C{" fields "}"
Actor       := "A[" id ":" role "]"
Actors      := Actor+
State       := "S{" fields "}"
Constraint  := "K[" predicate "]"
Constraints := Constraint*
Operation   := "O(" opcode ")"
Handler     := "H[" trigger "=>" Operation "]"
Handlers    := Handler*
Mediator    := "M(" Actor ")"
Scope       := "P{" domain "}"
Migration   := "G{" mapping+ "}"
Integrity   := "I[" invariant "]"
Variant     := "V[" source "+" delta "]"
Completion  := "Q[" predicate "]"
Transform   := State "--" Operation "-->" State
```

---

## 8. Minimal Executable Pattern

The smallest useful Scripture as Compute expression is:

```text
(C, A, S0) --[K] O--> S1
```

The smallest full resolution pattern is:

```text
(C, A, S0)
   --[K] O-->
S1
   --I-->
valid
   --Q-->
complete | continue
```

This is the current Core 0.1 transform syntax for Scripture as Compute.
