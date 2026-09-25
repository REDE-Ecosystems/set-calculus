# Scripture as Compute — Computer Science

## Status

The Core 0.1 runtime architecture is defined and the documentation branch is complete for review.

This does **not** claim that the runtime implementation, integration, or automated test suite already exists. Those are explicitly separated by the completion gates below.

## Core Documents

- [Core 0.1 Anchor](./ANCHOR_CORE_0.1.md)
- [Core Types and Transforms](./CORE_TYPES_AND_TRANSFORMS.md)
- [Typed Grammar](./TYPED_GRAMMAR.md)
- [AST and Parser](./AST_AND_PARSER.md)
- [Type Checker](./TYPE_CHECKER.md)
- [Interpreter and Execution Engine](./INTERPRETER_AND_EXECUTION_ENGINE.md)

## Start Runtime Library

- [Start Library Schema](./START_LIBRARY_SCHEMA.md)
- [Start Library Loading Pipeline](./START_LIBRARY_LOADING_PIPELINE.md)

The Start library defines the initial executable world and is converted into a typed runtime environment through parsing, import resolution, namespace binding, schema validation, dependency checking, type checking, and instantiation.

## Completion Gates

- [Runtime Completion Criteria](./RUNTIME_COMPLETION_CRITERIA.md)
- [Implementation Completeness Gate](./RUNTIME_IMPLEMENTATION_COMPLETENESS_GATE.md)
- [Integration Completeness Gate](./RUNTIME_INTEGRATION_COMPLETENESS_GATE.md)
- [Test Completeness Gate](./RUNTIME_TEST_COMPLETENESS_GATE.md)
- [Release Readiness Gate](./RUNTIME_RELEASE_READINESS_GATE.md)

The four completeness dimensions are intentionally independent:

```text
Architecture
Implementation
Integration
Tests
```

Release readiness requires all four gates to pass.

## Source Artifact

The original white-paper framing is preserved separately under:

```text
docs/scripture-as-compute/scripture-as-software-whitepaper.txt
```

That source is retained as provenance for the computational extraction.

## Core Model

A scriptural unit is modeled as an operation over contextual state:

```text
ScriptureUnit(Context, Actor, State) -> State'
```

Canonical typed form:

```text
(C:Context, A:Actor, S0:State)
    --[K:Constraint]
    O:Operation<State,State>
-->
S1:State
```

with post-transform evaluation:

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

## Execution Outcomes

```text
RESOLVED
UNRESOLVED
BLOCKED
INVALID
COMPLETE
```

## Runtime Architecture

```text
Start source
  -> parser
  -> import resolver
  -> namespace binder
  -> schema validator
  -> dependency checker
  -> type resolver
  -> type checker
  -> TypedStartGraph
  -> RuntimeEnvironment
  -> interpreter / execution engine
  -> scope / mediator / constraints
  -> operations / handlers
  -> migrations / variants
  -> integrity
  -> completion
  -> ExecutionResult
  -> execution trace + provenance
```

The runtime preserves immutable state transitions and source provenance throughout execution.

## Core 0.1 Boundary

The current architectural baseline is recorded in [ANCHOR_CORE_0.1.md](./ANCHOR_CORE_0.1.md).

The branch captures the state-machine/runtime architecture, Start library boundary, loading contract, execution semantics, and the concrete gates required to distinguish architectural completeness from implementation, integration, testing, and release readiness.

No further architectural work is required for this branch to be reviewed and merged. Future implementation work can proceed against the gates defined here.
