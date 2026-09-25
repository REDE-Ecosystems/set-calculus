# Scripture as Compute — Typed Grammar

## 1. Purpose

This document defines a typed grammar for Scripture as Compute.

The grammar formalizes the twelve core types:

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

The goal is to make Scripture as Compute expressions parseable, type-checkable, and executable.

---

## 2. Type Universe

```text
type State
type Context
type Actor
type Constraint
type Operation
type Handler
type Mediator
type Scope
type Migration
type Integrity
type Variant
type Completion
```

Primitive scalar types:

```text
type Bool
type Int
type String
type Symbol
type Time
type Identifier
```

Generic collection types:

```text
type List<T>
type Set<T>
type Map<K,V>
type Option<T>
```

---

## 3. Canonical Type Shapes

### State

```text
State := {
    id: Identifier,
    values: Map<Identifier, Value>,
    relations: Set<Relation>,
    status: StateStatus,
    provenance: List<Variant>
}
```

```text
StateStatus :=
    RESOLVED
  | UNRESOLVED
  | BLOCKED
  | INVALID
  | COMPLETE
```

---

### Context

```text
Context := {
    temporal: Option<Time>,
    linguistic: Option<Symbol>,
    cultural: Option<Symbol>,
    covenantal: Option<Symbol>,
    textual: Option<Symbol>,
    situational: Map<Identifier, Value>
}
```

---

### Actor

```text
Actor := {
    id: Identifier,
    role: Symbol,
    capabilities: Set<Symbol>,
    obligations: Set<Constraint>,
    permissions: Set<Symbol>,
    scope_membership: Set<Scope>
}
```

---

### Constraint

```text
Constraint := {
    predicate: Predicate,
    mode: ConstraintMode,
    target: Target,
    consequence: Option<Operation>
}
```

```text
ConstraintMode :=
    REQUIRE
  | FORBID
  | ALLOW
  | LIMIT
  | CONDITION
```

---

### Operation

```text
Operation<I,O> := {
    opcode: Symbol,
    input_type: I,
    output_type: O,
    preconditions: List<Constraint>,
    effect: Transform<I,O>
}
```

Canonical state operation:

```text
Operation<State, State>
```

---

### Handler

```text
Handler<E> := {
    trigger: E,
    guard: Option<Constraint>,
    operation: Operation<State,State>,
    fallback: Option<Operation<State,State>>
}
```

---

### Mediator

```text
Mediator := {
    actor: Actor,
    grants: Set<Symbol>,
    denies: Set<Symbol>,
    transforms: Set<Operation<State,State>>,
    protected_target: Target
}
```

---

### Scope

```text
Scope := {
    domain: Symbol,
    members: Set<Actor>,
    exclusions: Set<Actor>,
    boundary: Predicate
}
```

---

### Migration

```text
Migration<A,B> := {
    source_model: A,
    target_model: B,
    preserved: Set<Invariant>,
    replaced: Set<Mapping>,
    transformed: Set<Mapping>,
    deprecated: Set<Symbol>
}
```

---

### Integrity

```text
Integrity<T> := {
    invariant: Invariant<T>,
    verifier: Verify<T>,
    failure_condition: Predicate,
    provenance_rule: Option<Predicate>
}
```

Verifier signature:

```text
Verify<T> : T -> Bool
```

---

### Variant

```text
Variant<T> := {
    source: T,
    delta: Delta<T>,
    cause: Option<Symbol>,
    semantic_effect: Option<Transform<T,T>>,
    provenance: List<Identifier>
}
```

---

### Completion

```text
Completion<T> := {
    predicate: Predicate<T>,
    terminating_operation: Option<Operation<T,T>>,
    terminal_state: Option<T>
}
```

---

## 4. Scripture Unit Type

```text
ScriptureUnit := {
    context: Context,
    actors: List<Actor>,
    initial_state: State,
    constraints: List<Constraint>,
    operation: Operation<State,State>,
    handlers: List<Handler<Event>>,
    mediator: Option<Mediator>,
    scope: Scope,
    migration: Option<Migration<Model,Model>>,
    integrity: Option<Integrity<State>>,
    variants: List<Variant<TextArtifact>>,
    completion: Option<Completion<State>>
}
```

Execution signature:

```text
execute : ScriptureUnit -> ExecutionResult
```

```text
ExecutionResult := {
    state: State,
    status: StateStatus,
    trace: List<Transition>
}
```

---

## 5. Lexical Forms

Canonical short forms:

```text
S{...}        State
C{...}        Context
A[id:role]    Actor
K[...]        Constraint
O(...)        Operation
H[...]        Handler
M(...)        Mediator
P{...}        Scope
G{...}        Migration
I[...]        Integrity
V[...]        Variant
Q[...]        Completion
```

---

## 6. Typed Grammar

```ebnf
Program          := Declaration* UnitDecl* ;

Declaration      := StateDecl
                  | ContextDecl
                  | ActorDecl
                  | ConstraintDecl
                  | OperationDecl
                  | HandlerDecl
                  | MediatorDecl
                  | ScopeDecl
                  | MigrationDecl
                  | IntegrityDecl
                  | VariantDecl
                  | CompletionDecl ;

StateDecl        := "state" Identifier ":" "State" "=" StateExpr ;
ContextDecl      := "context" Identifier ":" "Context" "=" ContextExpr ;
ActorDecl        := "actor" Identifier ":" "Actor" "=" ActorExpr ;
ConstraintDecl   := "constraint" Identifier ":" "Constraint" "=" ConstraintExpr ;
OperationDecl    := "operation" Identifier ":" OperationType "=" OperationExpr ;
HandlerDecl      := "handler" Identifier ":" HandlerType "=" HandlerExpr ;
MediatorDecl     := "mediator" Identifier ":" "Mediator" "=" MediatorExpr ;
ScopeDecl        := "scope" Identifier ":" "Scope" "=" ScopeExpr ;
MigrationDecl    := "migration" Identifier ":" MigrationType "=" MigrationExpr ;
IntegrityDecl    := "integrity" Identifier ":" IntegrityType "=" IntegrityExpr ;
VariantDecl      := "variant" Identifier ":" VariantType "=" VariantExpr ;
CompletionDecl   := "completion" Identifier ":" CompletionType "=" CompletionExpr ;

OperationType    := "Operation" "<" Type "," Type ">" ;
HandlerType      := "Handler" "<" Type ">" ;
MigrationType    := "Migration" "<" Type "," Type ">" ;
IntegrityType    := "Integrity" "<" Type ">" ;
VariantType      := "Variant" "<" Type ">" ;
CompletionType   := "Completion" "<" Type ">" ;

UnitDecl         := "unit" Identifier ":" "ScriptureUnit" "=" UnitExpr ;

UnitExpr         := "{"
                       "context" ":" Ref ","
                       "actors" ":" "[" RefList? "]" ","
                       "initial_state" ":" Ref ","
                       "constraints" ":" "[" RefList? "]" ","
                       "operation" ":" Ref ","
                       "handlers" ":" "[" RefList? "]" ","
                       "mediator" ":" OptionalRef ","
                       "scope" ":" Ref ","
                       "migration" ":" OptionalRef ","
                       "integrity" ":" OptionalRef ","
                       "variants" ":" "[" RefList? "]" ","
                       "completion" ":" OptionalRef
                    "}" ;

TransformExpr    := StateRef Arrow OperationRef Arrow StateRef ;
GuardedTransform:= StateRef "--[" ConstraintRef "]" OperationRef "-->" StateRef ;
HandledTransform:= EventRef "->" HandlerRef "->" StateRef ;
MediatedTransform:= ActorRef "->" MediatorRef "->" OperationRef "->" StateRef ;
ScopedExpr       := ScopeRef "::" OperationRef ;
MigrationApply   := ModelRef "--" MigrationRef "-->" ModelRef ;
VariantApply     := ArtifactRef "--" VariantRef "-->" ArtifactRef ;
IntegrityCheck   := IntegrityRef "(" Expr ")" ;
CompletionCheck  := CompletionRef "(" Expr ")" ;

Arrow            := "--" ;
OptionalRef      := Ref | "none" ;
RefList          := Ref ("," Ref)* ;
Ref              := Identifier ;

Type             := "State"
                  | "Context"
                  | "Actor"
                  | "Constraint"
                  | "Operation"
                  | "Handler"
                  | "Mediator"
                  | "Scope"
                  | "Migration"
                  | "Integrity"
                  | "Variant"
                  | "Completion"
                  | "Bool"
                  | "Int"
                  | "String"
                  | "Symbol"
                  | "Time"
                  | Identifier ;
```

---

## 7. Expression Grammar

```ebnf
Expr             := Literal
                  | Ref
                  | FieldAccess
                  | Call
                  | Compare
                  | LogicalExpr
                  | TransformExpr
                  | IntegrityCheck
                  | CompletionCheck ;

FieldAccess      := Ref "." Identifier ;
Call             := Ref "(" ArgList? ")" ;
ArgList          := Expr ("," Expr)* ;

Compare          := Expr CompareOp Expr ;
CompareOp        := "==" | "!=" | "<" | ">" | "<=" | ">=" | "in" ;

LogicalExpr      := Expr LogicalOp Expr
                  | "not" Expr ;

LogicalOp        := "and" | "or" ;

Literal          := StringLiteral
                  | IntLiteral
                  | BoolLiteral
                  | SymbolLiteral ;
```

---

## 8. Static Typing Rules

### Rule T1: State transforms

```text
O : Operation<State,State>
S0 : State

therefore

O(S0) : State
```

---

### Rule T2: Constraint predicates

```text
K.predicate : Predicate
evaluate(K.predicate) : Bool
```

A Constraint cannot guard execution unless its predicate resolves to `Bool`.

---

### Rule T3: Handler compatibility

```text
H : Handler<E>
event : E
```

The Handler trigger type must match the event type.

---

### Rule T4: Mediator participation

For:

```text
A -> M -> O
```

the following must hold:

```text
A : Actor
M : Mediator
O : Operation<State,State>
M.protected_target accepts O
```

---

### Rule T5: Scope membership

If an Operation is scoped:

```text
P :: O
```

then every executing Actor must satisfy:

```text
P.boundary(A) == true
```

or execution resolves to:

```text
BLOCKED
```

---

### Rule T6: Migration compatibility

For:

```text
M0 --G--> M1
```

```text
G : Migration<TypeOf(M0), TypeOf(M1)>
```

The source and target model types must match the Migration type parameters.

---

### Rule T7: Integrity checks

```text
I : Integrity<T>
x : T
```

then:

```text
I(x) : Bool
```

A failed Integrity check produces:

```text
INVALID
```

unless explicitly handled.

---

### Rule T8: Variant source compatibility

```text
V : Variant<T>
x : T
```

therefore:

```text
apply(V,x) : T
```

A Variant preserves the outer type even when semantics differ.

---

### Rule T9: Completion compatibility

```text
Q : Completion<T>
x : T
```

then:

```text
Q(x) : Bool
```

If true, execution resolves to:

```text
COMPLETE
```

---

## 9. Typed Transform Form

The canonical typed Scripture as Compute expression is:

```text
(C:Context, A:Actor, S0:State)
    --[K:Constraint]
    O:Operation<State,State>
-->
S1:State
```

With verification and termination:

```text
I:Integrity<State>(S1)
Q:Completion<State>(S1)
```

Resolution:

```text
if not K(S0,A,C):
    BLOCKED

else:
    S1 = O(S0)

    if I exists and not I(S1):
        INVALID

    else if Q exists and Q(S1):
        COMPLETE

    else:
        RESOLVED
```

---

## 10. Typed Migration Form

```text
G : Migration<Model0,Model1>
```

Application:

```text
M0:Model0 --G--> M1:Model1
```

With preserved invariant:

```text
I : Integrity<SemanticContract>

I(contract(M0)) == true
I(contract(M1)) == true
```

This permits implementation-level change while preserving selected semantics.

---

## 11. Typed Variant Form

```text
V1 : Variant<TextArtifact>
V2 : Variant<TextArtifact>
```

Branching:

```text
T0:TextArtifact
+--V1--> T1:TextArtifact
+--V2--> T2:TextArtifact
```

Comparison:

```text
delta(T1,T2) : SemanticDelta
```

This provides the grammar needed for later provenance and textual-transmission analysis.

---

## 12. Core 0.1 Grammar Summary

The minimal typed program is:

```text
context C0 : Context = ...
actor A0 : Actor = ...
state S0 : State = ...
constraint K0 : Constraint = ...
operation O0 : Operation<State,State> = ...
scope P0 : Scope = ...
integrity I0 : Integrity<State> = ...
completion Q0 : Completion<State> = ...

unit U0 : ScriptureUnit = {
    context: C0,
    actors: [A0],
    initial_state: S0,
    constraints: [K0],
    operation: O0,
    handlers: [],
    mediator: none,
    scope: P0,
    migration: none,
    integrity: I0,
    variants: [],
    completion: Q0
}
```

Execution:

```text
execute(U0) : ExecutionResult
```

This is the typed grammar baseline for Scripture as Compute Core 0.1.
