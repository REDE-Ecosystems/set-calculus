# Scripture as Compute — Type Checker

## 1. Purpose

This document defines the type checker for the Scripture as Compute abstract syntax tree.

The type checker operates after parsing and binding:

```text
source
  -> lex
TokenStream
  -> parse
ProgramNode
  -> bind
BoundProgram
  -> typeCheck
TypedProgram
```

Its responsibilities are:

- symbol binding,
- duplicate detection,
- reference resolution,
- generic type resolution,
- operation signature checking,
- transform validation,
- migration validation,
- variant validation,
- integrity validation,
- completion validation,
- and production of a typed AST.

The type checker validates semantics. It does not execute Scripture as Compute programs.

---

# 2. Semantic Phases

The semantic pipeline is divided into four phases:

```text
1. collect declarations
2. bind references
3. resolve types
4. validate semantic rules
```

Expanded:

```text
ProgramNode
   ↓
SymbolCollection
   ↓
BoundProgram
   ↓
TypeResolution
   ↓
TypedProgram
   ↓
SemanticValidation
```

---

# 3. Symbol Table

## 3.1 Symbol

Every top-level declaration produces a Symbol.

```text
Symbol {
    name: Identifier
    kind: SymbolKind
    declared_type: TypeNode
    node: DeclarationNode
    span: SourceSpan
}
```

```text
SymbolKind :=
    STATE
  | CONTEXT
  | ACTOR
  | CONSTRAINT
  | OPERATION
  | HANDLER
  | MEDIATOR
  | SCOPE
  | MIGRATION
  | INTEGRITY
  | VARIANT
  | COMPLETION
  | UNIT
```

---

## 3.2 SymbolTable

```text
SymbolTable {
    symbols: Map<Identifier, Symbol>
}
```

Insertion:

```text
define(symbol)
```

Lookup:

```text
resolve(name) -> Symbol | UnresolvedSymbol
```

Duplicate declarations are rejected.

Example:

```text
state S0 : State = ...
state S0 : State = ...
```

produces:

```text
DuplicateSymbol("S0")
```

---

# 4. Symbol Collection

The first semantic pass walks all top-level declarations and units.

Pseudo-code:

```text
collectSymbols(program):
    table = SymbolTable()

    for declaration in program.declarations:
        symbol = symbolFromDeclaration(declaration)

        if table.contains(symbol.name):
            report DuplicateSymbol

        table.define(symbol)

    for unit in program.units:
        symbol = Symbol(
            name=unit.name,
            kind=UNIT,
            declared_type=ScriptureUnit,
            node=unit
        )

        if table.contains(symbol.name):
            report DuplicateSymbol

        table.define(symbol)

    return table
```

This phase does not resolve declaration bodies yet.

That permits forward references.

---

# 5. Binding

Binding replaces unresolved references with links to Symbols.

Input:

```text
RefNode("O0")
```

Bound representation:

```text
BoundRefNode {
    symbol: Symbol(O0)
}
```

If no symbol exists:

```text
UnresolvedReference("O0")
```

---

## 5.1 Binding Rules

For every `RefNode(name)`:

```text
symbol = SymbolTable.resolve(name)
```

If found:

```text
RefNode -> BoundRefNode(symbol)
```

If not found:

```text
TypeError.UnresolvedReference
```

---

## 5.2 Expected Symbol Kinds

Some AST locations constrain the legal symbol kind.

Examples:

```text
ScriptureUnit.context
    requires SymbolKind.CONTEXT

ScriptureUnit.operation
    requires SymbolKind.OPERATION

ScriptureUnit.scope
    requires SymbolKind.SCOPE
```

Mismatch:

```text
operation: C0
```

where `C0` is Context produces:

```text
ExpectedSymbolKind(
    expected=OPERATION,
    actual=CONTEXT
)
```

---

# 6. Type Representation

After parsing, types are converted into semantic types.

```text
SemanticType
```

Primary forms:

```text
PrimitiveType
NamedType
GenericType
TypeVariable
FunctionType
ErrorType
```

---

## 6.1 NamedType

```text
NamedType("State")
NamedType("Actor")
NamedType("Context")
```

---

## 6.2 GenericType

```text
GenericType {
    base: Identifier
    arguments: List<SemanticType>
}
```

Examples:

```text
Operation<State,State>
Integrity<State>
Variant<TextArtifact>
Migration<Model0,Model1>
Completion<State>
```

---

## 6.3 TypeVariable

Generic resolution uses type variables:

```text
T
I
O
A
B
E
```

Representation:

```text
TypeVariable("T")
```

---

# 7. Generic Type Resolution

Generic types are resolved by unification.

Example declaration:

```text
Operation<I,O>
```

Concrete use:

```text
Operation<State,State>
```

produces substitution:

```text
I := State
O := State
```

---

## 7.1 Unification

```text
unify(expected, actual, substitutions)
```

Rules:

### Identical named types

```text
unify(State, State) => success
```

### Type variable

```text
unify(T, State)
```

binds:

```text
T := State
```

### Generic types

```text
unify(
    Operation<T,T>,
    Operation<State,State>
)
```

recursively unifies arguments.

### Mismatch

```text
unify(State, Actor)
```

fails with:

```text
TypeMismatch(expected=State, actual=Actor)
```

---

## 7.2 Occurs Check

A type variable cannot resolve recursively to itself.

Invalid:

```text
T := Variant<T>
```

unless recursive types are explicitly introduced in a later version.

Core 0.1 rejects this.

---

# 8. Expression Type Checking

Every expression returns a semantic type.

```text
checkExpr(expr) -> SemanticType
```

Examples:

```text
Literal(true)       -> Bool
Literal(42)         -> Int
Ref(S0)             -> State
FieldAccess(A0.role)-> Symbol
```

---

# 9. Operation Type Checking

For:

```text
operation O0 : Operation<State,State> = ...
```

the declaration type is:

```text
Operation<State,State>
```

An invocation:

```text
O0(S0)
```

requires:

```text
type(S0) == State
```

and produces:

```text
State
```

General rule:

```text
O : Operation<I,O>
x : I

therefore

O(x) : O
```

---

# 10. Transform Validation

## 10.1 Basic Transform

AST:

```text
TransformNode {
    source,
    operation,
    target
}
```

Validation:

```text
sourceType = type(source)
operationType = type(operation)
targetType = type(target)
```

Require:

```text
operationType = Operation<I,O>
sourceType compatible with I
targetType compatible with O
```

Example:

```text
S0 --O0--> S1
```

valid when:

```text
S0 : State
O0 : Operation<State,State>
S1 : State
```

---

## 10.2 Guarded Transform

AST:

```text
GuardedTransformNode
```

Require:

```text
constraint : Constraint
operation : Operation<I,O>
source : I
target : O
```

Additionally:

```text
constraint.predicate : Bool
```

If the Constraint declaration carries a target type in a future extension, the target must be compatible with the transform source or Actor/Context domain.

Core 0.1 only requires Boolean predicate validity.

---

## 10.3 Handled Transform

For:

```text
event -> handler -> target
```

Require:

```text
handler : Handler<E>
event : E
handler.operation : Operation<State,State>
target : State
```

The Handler event generic must unify with the event expression type.

---

## 10.4 Mediated Transform

For:

```text
A0 -> M0 -> O0 -> S1
```

Require:

```text
A0 : Actor
M0 : Mediator
O0 : Operation<State,State>
S1 : State
```

And semantic compatibility:

```text
M0.actor accepts A0
M0.transforms contains or permits O0
M0.protected_target is compatible with O0 target
```

Core 0.1 permits this compatibility check to be structural rather than behavioral.

---

## 10.5 Scoped Transform

For:

```text
P0 :: O0
```

Require:

```text
P0 : Scope
O0 : Operation<I,O>
```

The operation remains typed:

```text
ScopedOperation<I,O>
```

Any executing Actor must later satisfy Scope membership during execution.

Static type checking validates only that the Scope and Operation types are legal.

---

# 11. ScriptureUnit Validation

For each `ScriptureUnitNode`, require:

```text
context       : Context
actors        : List<Actor>
initial_state : State
constraints   : List<Constraint>
operation     : Operation<State,State>
handlers      : List<Handler<Event>>
mediator      : Option<Mediator>
scope         : Scope
migration     : Option<Migration<*,*>>
integrity     : Option<Integrity<State>>
variants      : List<Variant<TextArtifact>>
completion    : Option<Completion<State>>
```

A field kind mismatch is a type error.

---

# 12. Migration Validation

Migration is a higher-order transform.

For:

```text
G : Migration<A,B>
```

application:

```text
X --G--> Y
```

requires:

```text
X : A
Y : B
```

Generic unification:

```text
unify(A, type(X))
unify(B, type(Y))
```

---

## 12.1 Migration Mapping Validation

A Migration contains mappings:

```text
source_property => target_property
```

Each mapping must reference legal properties of the source and target models.

Require:

```text
property(source_model, source_property) exists
property(target_model, target_property) exists
```

If property metadata is not available in Core 0.1, the mapping is preserved as symbolic and marked:

```text
UNRESOLVED_PROPERTY_MAPPING
```

rather than rejected solely for missing structural metadata.

---

## 12.2 Preserved Invariants

For every invariant in:

```text
G.preserved
```

require:

```text
Invariant<T>
```

to be compatible with both source and target semantic contract types.

If the invariant can only apply to one side, the migration is invalid.

---

# 13. Variant Validation

For:

```text
V : Variant<T>
```

require:

```text
V.source : T
V.delta : Delta<T>
```

Application:

```text
apply(V, x)
```

requires:

```text
x : T
```

and returns:

```text
T
```

Variant preserves outer type.

Example:

```text
Variant<TextArtifact>
```

may alter content while remaining:

```text
TextArtifact
```

---

## 13.1 Variant Semantic Effect

If:

```text
semantic_effect : Transform<T,T>
```

then require:

```text
input == T
output == T
```

A Variant may not silently change the outer type.

Invalid:

```text
Variant<TextArtifact>
semantic_effect : Transform<TextArtifact,State>
```

---

# 14. Integrity Validation

For:

```text
I : Integrity<T>
```

require:

```text
I.invariant : Invariant<T>
I.verifier : Verify<T>
```

with:

```text
Verify<T> : T -> Bool
```

Invocation:

```text
I(x)
```

requires:

```text
x : T
```

and produces:

```text
Bool
```

---

## 14.1 Integrity Failure Compatibility

If an integrity failure is handled:

```text
H[integrity_failure => O]
```

then:

```text
O : Operation<State,State>
```

for ScriptureUnit state validation.

A failed Integrity check without a Handler remains valid syntax and valid typing; it resolves dynamically to:

```text
INVALID
```

---

# 15. Completion Validation

For:

```text
Q : Completion<T>
```

require:

```text
Q.predicate : Predicate<T>
```

which means:

```text
T -> Bool
```

If:

```text
Q.terminating_operation
```

exists, require:

```text
Operation<T,T>
```

If:

```text
Q.terminal_state
```

exists, require:

```text
terminal_state : T
```

Invocation:

```text
Q(x)
```

requires:

```text
x : T
```

and produces:

```text
Bool
```

---

# 16. Constraint Validation

For every Constraint:

```text
predicate : Bool
```

If consequence exists:

```text
consequence : Operation<State,State>
```

Core 0.1 Constraint modes are:

```text
REQUIRE
FORBID
ALLOW
LIMIT
CONDITION
```

The mode must be one of these enumerated values.

---

# 17. Handler Validation

For:

```text
Handler<E>
```

require:

```text
trigger : E
guard : Option<Constraint>
operation : Operation<State,State>
fallback : Option<Operation<State,State>>
```

If both operation and fallback are present, both must return State.

---

# 18. Mediator Validation

Mediator fields:

```text
actor : Actor
grants : Set<Symbol>
denies : Set<Symbol>
transforms : Set<Operation<State,State>>
protected_target : Target
```

Type checker requirements:

```text
actor : Actor
transforms[*] : Operation<State,State>
```

Target compatibility may remain symbolic in Core 0.1 if Target is not yet structurally defined.

---

# 19. Scope Validation

Scope fields:

```text
domain : Symbol
members : Set<Actor>
exclusions : Set<Actor>
boundary : Predicate<Actor>
```

Require:

```text
members[*] : Actor
exclusions[*] : Actor
boundary : Actor -> Bool
```

An Actor present in both members and exclusions is semantically inconsistent.

Core 0.1 reports:

```text
ScopeMembershipConflict
```

---

# 20. Resolution Status Typing

The execution status enum is:

```text
StateStatus :=
    RESOLVED
  | UNRESOLVED
  | BLOCKED
  | INVALID
  | COMPLETE
```

Any State.status assignment must resolve to:

```text
StateStatus
```

Invalid:

```text
status = "happy"
```

unless explicitly represented as another field.

---

# 21. Type Checker Entry Point

```text
typeCheck(program: ProgramNode) -> TypedProgram
```

Pseudo-code:

```text
typeCheck(program):

    symbols = collectSymbols(program)

    bound = bind(program, symbols)

    resolveDeclaredTypes(bound)

    for declaration in bound.declarations:
        checkDeclaration(declaration)

    for unit in bound.units:
        checkScriptureUnit(unit)

    for expression in bound.expressions:
        checkExpr(expression)

    if diagnostics contain errors:
        return TypeErrorSet

    return TypedProgram(bound, symbols)
```

---

# 22. Typed AST

After successful validation, nodes carry resolved types.

Example:

```text
TypedRefNode {
    symbol: Symbol
    type: SemanticType
}
```

Transform:

```text
TypedTransformNode {
    source: TypedExpr<State>
    operation: TypedRef<Operation<State,State>>
    target: TypedExpr<State>
    type: State
}
```

Guarded transform:

```text
TypedGuardedTransformNode {
    source: TypedExpr<State>
    constraint: TypedRef<Constraint>
    operation: TypedRef<Operation<State,State>>
    target: TypedExpr<State>
    type: State
}
```

---

# 23. Type Errors

Minimum Core 0.1 diagnostics:

```text
DuplicateSymbol
UnresolvedReference
ExpectedSymbolKind
UnknownType
InvalidGenericArity
GenericTypeMismatch
TypeMismatch
InvalidOperationInput
InvalidOperationOutput
InvalidTransform
InvalidGuard
InvalidHandlerEvent
InvalidMediator
InvalidScope
InvalidMigration
InvalidMigrationMapping
InvalidVariantSource
InvalidVariantEffect
InvalidIntegrityTarget
InvalidIntegrityVerifier
InvalidCompletionTarget
InvalidCompletionPredicate
InvalidCompletionOperation
ScopeMembershipConflict
```

Each diagnostic carries:

```text
Diagnostic {
    code
    message
    span
    expected
    actual
    related_symbols
}
```

---

# 24. Minimal Example

Source:

```text
state S0 : State = ...
state S1 : State = ...

operation O0 : Operation<State,State> = ...

integrity I0 : Integrity<State> = ...
completion Q0 : Completion<State> = ...

S0 --O0--> S1
I0(S1)
Q0(S1)
```

Binding:

```text
S0 -> Symbol(State)
S1 -> Symbol(State)
O0 -> Symbol(Operation<State,State>)
I0 -> Symbol(Integrity<State>)
Q0 -> Symbol(Completion<State>)
```

Validation:

```text
type(S0) = State
type(O0) = Operation<State,State>
type(S1) = State

unify(Operation.input, State)  => success
unify(Operation.output, State) => success

type(I0) = Integrity<State>
I0(S1)   => Bool

type(Q0) = Completion<State>
Q0(S1)   => Bool
```

Result:

```text
TypedProgram
```

---

# 25. Minimal Generic Example

Declaration:

```text
integrity Preserve<T> : Integrity<T> = ...
```

Use:

```text
Preserve<State>(S1)
```

Resolution:

```text
T := State
```

Verifier becomes:

```text
State -> Bool
```

If used as:

```text
Preserve<Actor>(S1)
```

where:

```text
S1 : State
```

the checker reports:

```text
GenericTypeMismatch(
    expected=Actor,
    actual=State
)
```

---

# 26. Core 0.1 Semantic Contract

A Scripture as Compute program is type-valid when:

```text
1. every reference binds,
2. every declaration has a legal type,
3. every generic resolves consistently,
4. every transform matches operation input/output types,
5. every migration matches source and target model types,
6. every variant preserves its outer type,
7. every integrity verifier maps T -> Bool,
8. every completion predicate maps T -> Bool,
9. every ScriptureUnit field matches its required type,
10. no semantic consistency rule fails.
```

Formal front-end contract:

```text
typeCheck(bind(parse(source)))
    -> TypedProgram
```

or:

```text
TypeErrorSet
```

This defines the Scripture as Compute Core 0.1 type checker.
