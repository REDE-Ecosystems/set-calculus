# Scripture as Compute — Runtime Integration Completeness Gate

## 1. Purpose

This document defines the concrete integration-completeness gate for the Scripture as Compute runtime.

The integration gate answers one question:

> Do the implemented runtime modules compose correctly across the full Start-to-execution path without manual bridging, hidden translation layers, or broken provenance?

Integration completeness is distinct from:

```text
Architectural completeness
Implementation completeness
Test completeness
```

A runtime may have all required modules implemented and still fail integration if those modules do not compose correctly.

---

# 2. Integration Gate Definition

Integration is COMPLETE only when:

```text
CrossModuleWiringComplete
AND
EndToEndExecutionComplete
AND
DataContractContinuityComplete
AND
ErrorPropagationComplete
AND
TraceAndProvenanceContinuityComplete
AND
NoManualBridgingRequired
```

---

# 3. Required End-to-End Path

The following full path must execute successfully:

```text
.start source
   ↓
start_parser
   ↓
import_resolver
   ↓
namespace_binder
   ↓
schema_validator
   ↓
dependency_checker
   ↓
type_resolver
   ↓
type_checker
   ↓
runtime_environment_builder
   ↓
RuntimeEnvironment
   ↓
load ScriptureUnit
   ↓
scope_engine
   ↓
mediator_engine
   ↓
constraint_engine
   ↓
operation_engine
   ↓
handler_dispatcher
   ↓
migration_engine
   ↓
variant_engine
   ↓
integrity_engine
   ↓
completion_engine
   ↓
execution_engine
   ↓
ExecutionResult
   ↓
ExecutionTrace + Provenance
```

No step may require manual data rewriting or ad hoc conversion outside the defined interfaces.

---

# 4. Cross-Module Wiring Requirements

## 4.1 Parser -> Import Resolver

Input from parser:

```text
ParsedStartLibrary
```

must be accepted directly by:

```text
resolve_imports(...)
```

Acceptance:

- import declarations preserve source location
- import aliases survive parsing
- version constraints are available unchanged
- optional import flags survive parsing

Failure if:

- resolver requires reparsing source text
- resolver depends on parser-internal structures not part of the contract

---

## 4.2 Import Resolver -> Namespace Binder

Input:

```text
ResolvedStartGraph
```

Acceptance:

- imported library identity is preserved
- aliases are preserved
- include/exclude filters are preserved
- import graph edges remain available
- namespace binder does not rediscover imports independently

---

## 4.3 Namespace Binder -> Schema Validator

Input:

```text
BoundStartGraph
```

Acceptance:

- every declaration has a qualified symbol
- every reference retains original source span
- unresolved references remain explicit diagnostics
- schema validation reads bound structures directly

---

## 4.4 Schema Validator -> Dependency Checker

Input:

```text
SchemaValidStartGraph
```

Acceptance:

- declaration families are normalized
- required fields are known present
- dependency checker does not repeat structural validation
- optional fields remain distinguishable from missing required fields

---

## 4.5 Dependency Checker -> Type Resolver

Input:

```text
DependencyValidStartGraph
```

Acceptance:

- dependency references are resolved to bound symbols
- dependency kinds are known
- type resolver consumes graph without symbol rebinding
- legal cycles remain represented

---

## 4.6 Type Resolver -> Type Checker

Input:

```text
TypeResolvedStartGraph
```

Acceptance:

- generic arguments are resolved
- type variables/substitutions are preserved
- type checker consumes semantic types directly
- no type expression is reparsed from text

---

## 4.7 Type Checker -> Runtime Environment Builder

Input:

```text
TypedStartGraph
```

Acceptance:

- every declaration has resolved semantic type
- bound symbol identity is preserved
- runtime builder does not perform type checking again
- declaration provenance is retained

---

## 4.8 Runtime Environment Builder -> Execution Engine

Input:

```text
RuntimeEnvironment
```

Acceptance:

- contexts are addressable
- actors are addressable
- states are addressable
- scopes are addressable
- constraints are addressable
- mediators are addressable
- operations are executable
- handlers are registered
- migrations are registered
- variants are registered
- integrity rules are executable
- completions are executable

---

# 5. Runtime Wiring Requirements

## 5.1 Scope -> Execution

```text
execution_engine
    -> scope_engine
```

Acceptance:

- actors passed by execution engine are the same runtime actor identities produced by the loader
- scope decisions affect execution status directly
- scope denial returns BLOCKED
- no duplicate scope-resolution path exists elsewhere

---

## 5.2 Mediator -> Operation Path

```text
execution_engine
    -> mediator_engine
    -> operation_engine
```

Acceptance:

- mediator receives actual RuntimeOperation handle
- mediator grants/denials affect that operation
- mediator-produced state becomes operation input
- denial prevents operation execution

---

## 5.3 Constraints -> Operations

```text
constraint_engine
    -> operation_engine
```

Acceptance:

- preconditions use shared runtime evaluation context
- consequence operations use the same operation executor
- blocked constraints prevent primary operation execution
- consequence state is returned into execution flow

---

## 5.4 Operations -> Handlers

```text
operation_engine
    -> emitted events
    -> handler_dispatcher
```

Acceptance:

- events emitted by operations are delivered without translation
- event type identity is preserved
- handler priority ordering is honored
- handler-produced state becomes subsequent runtime state

---

## 5.5 Handlers -> Runtime State

Acceptance:

```text
S0 -> O -> S1 -> H1 -> S2 -> H2 -> S3
```

must preserve:

- version sequence
- provenance sequence
- trace order
- source operation identity

---

## 5.6 Migration -> Live Runtime Model

```text
execution_engine
    -> migration_engine
```

Acceptance:

- migration consumes live runtime model/state
- source model type matches registry metadata
- migration mappings affect runtime representation
- preserved invariants are checked before and after
- migration result becomes current execution model/state

---

## 5.7 Variant -> Runtime Artifact/State

```text
execution_engine
    -> variant_engine
```

Acceptance:

- variant selection receives current context
- selection policy is applied once
- selected variant preserves target outer type
- applied variant provenance is appended
- resulting artifact/state flows into integrity check

---

## 5.8 Integrity -> Handler Repair

```text
integrity_engine
    -> integrity_failed event
    -> handler_dispatcher
    -> repaired state
    -> integrity_engine recheck
```

Acceptance:

- repair uses normal handler dispatch
- repaired state is not special-cased
- exactly one automatic recheck occurs
- failed repair returns INVALID

---

## 5.9 Completion -> Execution Termination

```text
completion_engine
    -> execution_engine
```

Acceptance:

- completion predicate sees final post-integrity state
- terminating operation uses normal operation engine
- terminal state preserves provenance
- COMPLETE terminates unit execution cleanly

---

# 6. Shared Runtime Context Requirement

All runtime modules must operate against a compatible shared context:

```text
RuntimeExecutionContext {
    environment
    current_state
    context
    actors
    current_unit
    event_queue
    trace
    provenance
    diagnostics
}
```

Acceptance:

- modules do not create isolated duplicate environments
- trace/provenance references point to the same execution instance
- state identity is consistent across modules

---

# 7. Data Contract Continuity

No module may silently discard required fields.

The following must survive across the full pipeline:

```text
qualified symbol
source library
source span
library version
schema version
semantic type
runtime object identity
state version
provenance chain
trace sequence
diagnostic code
```

---

# 8. Identifier Continuity

Example:

```text
sinai::Obey
```

must remain identifiable as the same declaration through:

```text
Parsed declaration
-> BoundSymbol
-> TypedOperationDecl
-> RuntimeOperation
-> TraceEvent
-> ProvenanceEntry
```

Acceptance:

```text
origin(RuntimeOperation) == sinai::Obey
origin(TraceEvent.operation) == sinai::Obey
```

---

# 9. State Continuity

For a transition:

```text
S0 -> O -> S1
```

integration requires:

```text
S1.version == S0.version + 1
S1.provenance includes O
trace contains STATE_TRANSITION(S0,S1,O)
S0 remains available
```

No module may mutate S0 in place.

---

# 10. Diagnostic Propagation

Diagnostics must cross module boundaries without being flattened into generic errors.

Example:

```text
scope_engine
    -> ScopeBlocked
    -> execution_engine
    -> ExecutionResult.diagnostics
```

Acceptance:

- original diagnostic code survives
- source span survives
- related symbol survives
- execution engine may wrap but not erase root cause

---

# 11. Failure Short-Circuiting

Some failures must stop later phases.

Examples:

```text
parse failure
    => no import resolution

schema failure
    => no runtime instantiation

scope BLOCKED
    => no primary operation

mediator BLOCKED
    => no primary operation

integrity INVALID after repair
    => no completion evaluation
```

Acceptance:

- forbidden later phases do not run
- trace confirms they did not run

---

# 12. End-to-End Positive Scenario

Minimum integrated success case:

```text
Start library:
    Context C0
    Actor A0
    State S0
    Scope P0
    Constraint K0
    Operation O0
    Handler H0
    Integrity I0
    Completion Q0
```

Expected execution:

```text
load Start
    ↓
RuntimeEnvironment

execute Unit
    ↓
scope passes
    ↓
constraint passes
    ↓
O0 executes
    ↓
event emitted
    ↓
H0 dispatches
    ↓
integrity passes
    ↓
completion passes
    ↓
COMPLETE
```

Acceptance:

```text
final status == COMPLETE
final state version >= 1
trace contains every executed phase
provenance resolves to Start declarations
```

---

# 13. End-to-End Import Scenario

Minimum imported-library scenario:

```text
root.start
    imports common.start as common

root Actor
    references common::Scope
```

Acceptance:

- import resolves
- namespace binds
- dependency checker accepts qualified reference
- runtime registry contains both namespaces
- execution uses imported runtime object successfully
- provenance records import path

---

# 14. End-to-End Mediated Scenario

```text
Actor A
Scope P
Mediator M
Operation O
```

Expected:

```text
A allowed by P
M permits A + O
M transform applied
O executes
result continues
```

Negative counterpart:

```text
M denies O
=> BLOCKED
=> O does not execute
```

---

# 15. End-to-End Migration Scenario

```text
ModelA
Migration G
ModelB
```

Acceptance:

- migration registry resolves G
- source type matches
- mappings apply
- preserved invariant checked before and after
- target runtime model produced
- execution continues using ModelB
- trace links G to source declaration

---

# 16. End-to-End Variant Scenario

```text
Variant V1
Variant V2
policy = FIRST_MATCH
```

Acceptance:

- context selects V1
- V2 is not applied
- outer type preserved
- provenance includes V1
- integrity receives variant-modified target

---

# 17. End-to-End Integrity Repair Scenario

Expected flow:

```text
integrity fails
   ↓
integrity_failed event
   ↓
repair handler
   ↓
new state
   ↓
integrity recheck
   ↓
pass
   ↓
completion evaluation
```

Acceptance:

- exactly one repair cycle
- trace shows both checks
- repaired state provenance includes handler operation

---

# 18. End-to-End Unresolved Scenario

A missing runtime value causes:

```text
UNKNOWN
```

Acceptance:

- UNKNOWN propagates through TriBool
- required unresolved condition produces UNRESOLVED
- false is not substituted for unknown
- trace identifies unresolved expression

---

# 19. End-to-End Failure Scenario

At least one full negative flow must demonstrate:

```text
valid Start load
    ↓
execution begins
    ↓
runtime rule fails
    ↓
structured diagnostic
    ↓
correct final status
    ↓
trace ends at failure point
```

Example:

```text
scope exclusion
=> BLOCKED
```

---

# 20. Integration Acceptance Criteria

Integration is COMPLETE when all of the following are true:

```text
G1. Parser output feeds import resolver directly.
G2. Import resolver output feeds namespace binder directly.
G3. Bound graph feeds schema validator directly.
G4. Schema-valid graph feeds dependency checker directly.
G5. Dependency-valid graph feeds type resolver/checker directly.
G6. Typed graph feeds RuntimeEnvironment builder directly.
G7. RuntimeEnvironment feeds execution engine directly.
G8. Scope decisions affect execution.
G9. Mediator decisions affect execution.
G10. Constraint decisions affect execution.
G11. Operation-emitted events reach handlers.
G12. Handler state transitions feed subsequent phases.
G13. Migrations modify live execution state/model.
G14. Variant selection/application affects the integrity target.
G15. Integrity repair routes through normal handler dispatch.
G16. Completion terminates execution correctly.
G17. State version/provenance remain continuous.
G18. Trace ordering matches actual execution ordering.
G19. Diagnostics retain original module/source information.
G20. Imports and qualified symbols survive into runtime.
G21. No manual data translation is required between stages.
G22. No duplicate semantic path bypasses the defined modules.
```

---

# 21. Required Integration Evidence

For every integration boundary, record:

```text
Boundary:
Producer:
Consumer:
Input type:
Output type:
Observed handoff:
Manual conversion required: yes/no
Result:
```

Example:

```text
Boundary: Type Checker -> Runtime Builder
Producer: type_checker
Consumer: runtime_environment_builder
Input type: TypedStartGraph
Output type: RuntimeEnvironment
Observed handoff: direct
Manual conversion required: no
Result: pass
```

---

# 22. Integration Checklist

```text
[ ] parser -> import resolver wired
[ ] import resolver -> namespace binder wired
[ ] namespace binder -> schema validator wired
[ ] schema validator -> dependency checker wired
[ ] dependency checker -> type resolver wired
[ ] type resolver -> type checker wired
[ ] type checker -> runtime builder wired
[ ] runtime builder -> execution engine wired

[ ] execution -> scope engine wired
[ ] execution -> mediator engine wired
[ ] execution -> constraint engine wired
[ ] execution -> operation engine wired
[ ] operation events -> handler dispatcher wired
[ ] execution -> migration engine wired
[ ] execution -> variant engine wired
[ ] execution -> integrity engine wired
[ ] integrity failure -> handler repair wired
[ ] execution -> completion engine wired

[ ] state version continuity preserved
[ ] provenance continuity preserved
[ ] trace continuity preserved
[ ] diagnostic continuity preserved
[ ] namespace identity preserved
[ ] imported symbol identity preserved

[ ] minimal positive end-to-end scenario passes
[ ] imported-library end-to-end scenario passes
[ ] mediated scenario passes
[ ] migration scenario passes
[ ] variant scenario passes
[ ] integrity repair scenario passes
[ ] unresolved scenario passes
[ ] negative end-to-end scenario passes

[ ] no manual bridge code required
[ ] no duplicate bypass path exists
```

---

# 23. Pass / Fail Rule

Integration status is COMPLETE only if:

```text
all required module boundaries are wired
AND
all required runtime phase boundaries are wired
AND
all required end-to-end scenarios pass
AND
state/provenance/trace/diagnostics remain continuous
AND
no manual bridge is required
AND
no semantic bypass path exists
```

Otherwise:

```text
IntegrationStatus = PARTIAL
```

---

# 24. Boundary With Test Completeness

Integration completeness proves:

```text
the system composes correctly across required paths
```

Test completeness proves:

```text
those behaviors are repeatably guarded across required positive and negative cases
```

A runtime may be integration-complete before the full automated test matrix is complete.

---

# 25. Integration Done Definition

The runtime is integration-complete when:

> A valid Start library can flow through parsing, imports, namespaces, schema validation, dependency checks, type resolution, runtime construction, and ScriptureUnit execution without manual translation, while preserving symbol identity, state continuity, provenance, diagnostics, and trace semantics across every required runtime phase.

This defines the concrete integration-completeness gate for the Scripture as Compute runtime.
