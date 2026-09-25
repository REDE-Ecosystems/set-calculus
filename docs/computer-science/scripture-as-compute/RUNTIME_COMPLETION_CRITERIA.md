# Scripture as Compute — Runtime Completion Criteria

## 1. Purpose

This document defines concrete completion criteria for the Scripture as Compute runtime.

Completion is divided into four independent dimensions:

```text
Architectural Completeness
Implementation Completeness
Integration Completeness
Test Completeness
```

These dimensions must not be conflated.

A runtime may be architecturally complete while implementation, integration, or testing remain incomplete.

---

# 2. Completion Model

```text
RuntimeCompletion := {
    architecture: Status,
    implementation: Status,
    integration: Status,
    tests: Status
}
```

```text
Status :=
    NOT_STARTED
  | PARTIAL
  | COMPLETE
```

Overall production readiness requires:

```text
architecture   == COMPLETE
implementation == COMPLETE
integration    == COMPLETE
tests          == COMPLETE
```

Architectural readiness requires only:

```text
architecture == COMPLETE
```

---

# 3. Architectural Completeness

Architectural completeness means the runtime has no unresolved conceptual component required to define how execution works.

It does not require executable code.

## 3.1 Required architectural artifacts

The following must exist and be internally consistent:

- Core runtime types
- Typed grammar
- AST definition
- Parser model
- Symbol binding model
- Generic type-resolution rules
- Type checker
- Start library schema
- Start loading pipeline
- Runtime environment model
- Runtime state model
- Execution result model
- Execution trace model
- Scope enforcement rules
- Mediator rules
- Constraint evaluation rules
- Operation execution rules
- Handler dispatch rules
- Migration execution rules
- Variant selection/application rules
- Integrity validation rules
- Completion evaluation rules
- Runtime status-resolution rules
- Three-valued logic for unresolved values
- Provenance model
- Failure/diagnostic model

## 3.2 Architectural pass criteria

Architecture is COMPLETE when all of the following are true:

```text
A1. Every runtime concept has a defined type or contract.
A2. Every execution phase has a defined input and output.
A3. Execution order is explicitly defined.
A4. Start loading has a deterministic staged pipeline.
A5. Runtime state transition semantics are defined.
A6. Error/status outcomes are defined.
A7. Scope, mediator, constraint, handler, migration, variant,
    integrity, and completion semantics are defined.
A8. Provenance is preserved across loading and execution.
A9. No required phase depends on an undefined conceptual object.
A10. No known architectural contradiction remains unresolved.
```

## 3.3 Architectural failure criteria

Architecture is not complete if any of these remain:

- undefined runtime phase
- ambiguous execution order
- missing type contract
- unresolved ownership of state mutation
- undefined failure semantics
- undefined library/runtime boundary
- undefined provenance behavior
- undefined termination/completion semantics
- conflicting rules between loader and runtime

---

# 4. Implementation Completeness

Implementation completeness means the defined architecture exists as working code.

It does not require external integration or full test coverage.

## 4.1 Required implemented components

At minimum:

```text
Start parser
Import resolver
Namespace binder
Schema validator
Dependency checker
Type resolver
Type checker
Runtime environment builder
Runtime state instantiator
Expression evaluator
Scope evaluator
Mediator evaluator
Constraint evaluator
Operation executor
Handler dispatcher
Migration executor
Variant selector
Integrity evaluator
Completion evaluator
Execution trace recorder
Diagnostic emitter
```

## 4.2 Implementation pass criteria

Implementation is COMPLETE when:

```text
I1. Every architectural runtime phase has executable code.
I2. Start libraries can be loaded into a RuntimeEnvironment.
I3. A valid ScriptureUnit can execute from S0 to S1.
I4. Runtime statuses are emitted correctly.
I5. State transitions preserve prior state.
I6. Execution trace records every required phase.
I7. Runtime objects retain provenance.
I8. Invalid inputs fail with structured diagnostics.
I9. No required architectural component is represented only by a stub.
I10. The implementation builds/runs without manual patching.
```

## 4.3 Implementation is still partial if

- functions are placeholders
- handlers are declared but not dispatchable
- migrations are parsed but not executable
- variants are defined but not selectable
- integrity checks are skipped
- completion rules are ignored
- provenance fields are not populated
- runtime diagnostics are strings instead of structured results
- Start imports only work in a hard-coded case

---

# 5. Integration Completeness

Integration completeness means the runtime components work together through the complete path.

This is broader than each component working independently.

## 5.1 Required integration path

The full path must work:

```text
.start source
   ↓
parse
   ↓
resolve imports
   ↓
bind namespaces
   ↓
validate schema
   ↓
check dependencies
   ↓
resolve/check types
   ↓
instantiate RuntimeEnvironment
   ↓
load ScriptureUnit
   ↓
execute
   ↓
trace + provenance + result
```

## 5.2 Integration pass criteria

Integration is COMPLETE when:

```text
G1. Start source can flow end-to-end into runtime execution.
G2. Imported libraries resolve across namespaces.
G3. Qualified references survive into runtime handles.
G4. Typed declarations instantiate correctly.
G5. Runtime execution consumes objects produced by the loader
    without translation hacks.
G6. Handler dispatch works with emitted runtime events.
G7. Mediator and scope rules affect actual execution.
G8. Migrations operate on real runtime models.
G9. Variants selected by policy affect execution state or artifacts.
G10. Integrity checks run against post-transform results.
G11. Completion evaluation terminates execution correctly.
G12. Trace and provenance link back to source declarations.
```

## 5.3 Integration failure examples

Integration is incomplete if:

- loader output must be manually rewritten for runtime use
- namespace-qualified imports lose identity
- type information is discarded before execution
- handlers cannot receive runtime events
- migration objects exist but cannot alter live execution
- variants are not visible to runtime selection
- trace entries cannot identify originating declarations
- Start defaults do not initialize execution correctly

---

# 6. Test Completeness

Test completeness means the runtime behavior is demonstrated and guarded by repeatable tests.

It is not equivalent to merely having tests.

## 6.1 Required test classes

At minimum:

```text
unit tests
parser tests
schema tests
binding tests
type-check tests
dependency tests
runtime transition tests
handler tests
mediator tests
scope tests
migration tests
variant tests
integrity tests
completion tests
trace/provenance tests
end-to-end tests
failure-path tests
cycle-detection tests
```

## 6.2 Test pass criteria

Tests are COMPLETE when:

```text
T1. Every runtime phase has at least one positive test.
T2. Every runtime phase has at least one failure-path test.
T3. All five execution statuses are exercised:
    RESOLVED
    UNRESOLVED
    BLOCKED
    INVALID
    COMPLETE
T4. Imported Start libraries are tested.
T5. Namespace collisions are tested.
T6. Missing dependencies are tested.
T7. Generic mismatches are tested.
T8. Handler dispatch order is tested.
T9. Mediator denial is tested.
T10. Scope exclusion precedence is tested.
T11. Migration invariant failure is tested.
T12. Variant selection policies are tested.
T13. Integrity repair/recheck behavior is tested.
T14. Completion termination is tested.
T15. Execution-cycle detection is tested.
T16. Provenance survives an end-to-end run.
T17. At least one complete Start → RuntimeEnvironment →
     ScriptureUnit → ExecutionResult flow passes.
```

## 6.3 Test completeness does not require

Core 0.1 does not require:

- exhaustive theological validation
- exhaustive corpus coverage
- performance benchmarking
- production-scale load testing
- formal proof of semantic correctness

Those may be separate future validation tracks.

---

# 7. Completion Matrix

```text
+----------------------+---------------------------------------------+
| Dimension            | Complete when                               |
+----------------------+---------------------------------------------+
| Architecture         | All runtime concepts and contracts defined  |
| Implementation       | All required runtime components executable  |
| Integration          | Full source-to-execution path works         |
| Tests                | Required behavior and failure paths covered |
+----------------------+---------------------------------------------+
```

---

# 8. Current Status Interpretation

The current documentation can support:

```text
Architecture: COMPLETE
```

provided no contradictory requirement is discovered during implementation.

The following are separate and must not be inferred from architectural completeness:

```text
Implementation: not established by documentation alone
Integration:    not established by documentation alone
Tests:          not established by documentation alone
```

---

# 9. Runtime Done Definitions

## Architecturally done

```text
There is nothing essential left to define before implementation can proceed.
```

## Implementation done

```text
Every defined runtime component exists as executable code.
```

## Integration done

```text
The complete loader-to-runtime execution path works without manual bridging.
```

## Test done

```text
The defined runtime behavior and failure modes are repeatably demonstrated.
```

## Fully complete

```text
architecture   == COMPLETE
implementation == COMPLETE
integration    == COMPLETE
tests          == COMPLETE
```

This defines the concrete runtime completion criteria for Scripture as Compute Core 0.1.
