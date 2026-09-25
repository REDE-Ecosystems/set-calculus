# Scripture as Compute — Core 0.1 Anchor

## Anchor Purpose

This file marks the current stable conceptual boundary of the Scripture as Compute branch.

Everything below this point has been defined at the model level and should be treated as the Core 0.1 baseline unless explicitly revised.

## Core 0.1 Stack

```text
Source model
→ typed grammar
→ AST
→ parser
→ symbol binding
→ generic type resolution
→ type checker
→ typed AST
→ interpreter
→ runtime state transitions
→ handler / mediator / scope logic
→ migration / variant execution
→ integrity checks
→ completion evaluation
→ execution trace + provenance
```

## Canonical Transform

```text
(C:Context, A:Actor, S0:State)
    --[K:Constraint]
    O:Operation<State,State>
-->
S1:State
```

Post-transform evaluation:

```text
I:Integrity<State>(S1)
Q:Completion<State>(S1)
```

## Core Types

```text
State
Context
Actor
Constraint
Operation
Handler
Mediator
Scope
Migration
Integrity
Variant
Completion
```

## Runtime Outcomes

```text
RESOLVED
UNRESOLVED
BLOCKED
INVALID
COMPLETE
```

## Runtime Order

```text
1. load typed unit
2. load runtime state/context/actors
3. enforce scope
4. apply mediator
5. evaluate constraints
6. execute primary operation
7. dispatch handlers
8. execute migration if present
9. select/apply variants
10. validate integrity
11. evaluate completion
12. return state + status + trace
```

## Front-End Contract

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

## Runtime Contract

```text
interpret(
    typedProgram,
    scriptureUnit,
    executionInput
)
    -> ExecutionResult
```

## Provenance Principle

Runtime state transitions are preserved rather than overwritten:

```text
S0
 -> S1
 -> S2
 -> ...
```

Each transition contributes to an execution trace describing which operation, constraint, mediator, handler, migration, variant, integrity rule, or completion rule produced the resulting state.

## Anchor Statement

As of Core 0.1, Scripture as Compute is defined as a typed executable transformation model in which a scriptural unit operates over contextual state, under actors, constraints, scope, mediation, handlers, migration, integrity, variants, and completion semantics.

This anchor exists to capture the current system state. It does not establish a roadmap, priority order, or requirement to continue the compiler/runtime stack.
