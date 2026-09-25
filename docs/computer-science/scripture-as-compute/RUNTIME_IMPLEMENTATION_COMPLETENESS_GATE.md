# Scripture as Compute — Runtime Implementation Completeness Gate

## 1. Purpose

This document defines the concrete implementation-completeness gate for the Scripture as Compute runtime.

The implementation gate answers one question:

> Does every architecturally required runtime component exist as executable code behind a stable interface?

This gate does not certify end-to-end integration or test completeness.

---

# 2. Gate Definition

Implementation is COMPLETE only when all required modules exist, expose the required interfaces, satisfy the acceptance criteria, and contain no required stubs.

```text
ImplementationComplete :=
    RequiredModulesPresent
  ∧ RequiredInterfacesImplemented
  ∧ RequiredAcceptanceCriteriaPassed
  ∧ NoRequiredStubs
```

---

# 3. Required Module Set

The runtime implementation must include the following modules.

```text
start_parser
import_resolver
namespace_binder
schema_validator
dependency_checker
type_resolver
type_checker
runtime_environment_builder
runtime_state_factory
expression_evaluator
scope_engine
mediator_engine
constraint_engine
operation_engine
handler_dispatcher
migration_engine
variant_engine
integrity_engine
completion_engine
execution_engine
trace_recorder
provenance_engine
diagnostics
```

---

# 4. Module Interfaces

## 4.1 start_parser

Responsibility:

Parse Start source into a syntax tree.

Interface:

```text
parse_start(source: StartSource)
    -> ParsedStartLibrary
    | ParseDiagnosticSet
```

Acceptance:

- valid Start source produces a ParsedStartLibrary
- malformed syntax produces structured parse diagnostics
- no semantic resolution occurs during parsing

---

## 4.2 import_resolver

Responsibility:

Resolve imported Start libraries and construct the import graph.

Interface:

```text
resolve_imports(
    root: ParsedStartLibrary,
    options: ImportResolutionOptions
)
    -> ResolvedStartGraph
    | ImportDiagnosticSet
```

Acceptance:

- relative imports resolve
- search-path imports resolve
- aliases are preserved
- missing imports produce diagnostics
- import cycles are detected

---

## 4.3 namespace_binder

Responsibility:

Bind declarations and references to fully qualified symbols.

Interface:

```text
bind_namespaces(
    graph: ResolvedStartGraph
)
    -> BoundStartGraph
    | NamespaceDiagnosticSet
```

Acceptance:

- local symbols resolve
- qualified symbols resolve
- aliases resolve
- ambiguous references fail
- duplicate qualified symbols fail

---

## 4.4 schema_validator

Responsibility:

Validate Start declaration structure.

Interface:

```text
validate_start_schema(
    graph: BoundStartGraph
)
    -> SchemaValidStartGraph
    | SchemaDiagnosticSet
```

Acceptance:

- required metadata fields are enforced
- unsupported schema versions fail
- each declaration family enforces required fields
- optional fields remain optional

---

## 4.5 dependency_checker

Responsibility:

Validate declaration references and dependency kinds.

Interface:

```text
check_dependencies(
    graph: SchemaValidStartGraph
)
    -> DependencyValidStartGraph
    | DependencyDiagnosticSet
```

Acceptance:

- missing dependencies fail
- wrong-kind dependencies fail
- legal reference cycles are allowed
- illegal instantiation cycles fail
- default state/context/scope references are validated

---

## 4.6 type_resolver

Responsibility:

Resolve named and generic type expressions into semantic types.

Interface:

```text
resolve_types(
    graph: DependencyValidStartGraph
)
    -> TypeResolvedStartGraph
    | TypeDiagnosticSet
```

Acceptance:

- named types resolve
- generic arguments resolve
- invalid generic arity fails
- unresolved type names fail
- type-variable substitutions are retained

---

## 4.7 type_checker

Responsibility:

Validate declaration and transform type compatibility.

Interface:

```text
type_check_start(
    graph: TypeResolvedStartGraph
)
    -> TypedStartGraph
    | TypeDiagnosticSet
```

Acceptance:

- Operation<I,O> input/output are enforced
- Handler<E> event types are enforced
- Migration<A,B> source/target are enforced
- Integrity<T> verifier returns Bool
- Completion<T> predicate returns Bool
- Variant<T> preserves outer type

---

## 4.8 runtime_environment_builder

Responsibility:

Convert a TypedStartGraph into runtime registries.

Interface:

```text
build_runtime_environment(
    graph: TypedStartGraph,
    options?: RuntimeBuildOptions
)
    -> RuntimeEnvironment
    | RuntimeBuildDiagnosticSet
```

Acceptance:

- every typed declaration family is registered
- qualified symbol identity is preserved
- runtime objects retain source provenance
- partial environments are not returned on failure

---

## 4.9 runtime_state_factory

Responsibility:

Instantiate immutable runtime State values.

Interface:

```text
instantiate_state(
    declaration: TypedStateDecl
)
    -> RuntimeState
```

Acceptance:

- initial version is 0
- source declaration remains immutable
- runtime values are copied correctly
- provenance contains source declaration identity

---

## 4.10 expression_evaluator

Responsibility:

Evaluate typed runtime expressions.

Interface:

```text
eval_expr(
    expr: TypedExpr,
    env: RuntimeEvaluationContext
)
    -> RuntimeValue
    | RuntimeDiagnostic
```

Acceptance:

- literals evaluate
- references evaluate
- field access evaluates
- comparisons evaluate
- logical expressions use TriBool
- UNKNOWN propagates according to defined three-valued logic

---

## 4.11 scope_engine

Responsibility:

Enforce runtime Scope membership.

Interface:

```text
evaluate_scope(
    scope: RuntimeScope,
    actor: RuntimeActor,
    env: RuntimeEvaluationContext
)
    -> ScopeDecision
```

Acceptance:

- explicit exclusion wins
- explicit membership permits
- boundary predicate is evaluated
- unresolved boundary produces UNRESOLVED
- denial produces BLOCKED semantics

---

## 4.12 mediator_engine

Responsibility:

Apply mediator permissions and transforms.

Interface:

```text
apply_mediator(
    mediator: RuntimeMediator,
    actor: RuntimeActor,
    operation: RuntimeOperation,
    state: RuntimeState,
    env: RuntimeExecutionContext
)
    -> MediatorResult
```

Acceptance:

- actor eligibility is checked
- grants/denials are enforced
- protected operation compatibility is checked
- mediator transforms can produce a new state
- denial produces BLOCKED

---

## 4.13 constraint_engine

Responsibility:

Evaluate runtime Constraints.

Interface:

```text
evaluate_constraint(
    constraint: RuntimeConstraint,
    env: RuntimeExecutionContext,
    state: RuntimeState
)
    -> ConstraintResult
```

Acceptance:

- REQUIRE
- FORBID
- ALLOW
- LIMIT
- CONDITION

must each have executable behavior.

If a blocking constraint has a consequence Operation, the consequence must be invokable.

---

## 4.14 operation_engine

Responsibility:

Execute Operation<State,State>.

Interface:

```text
execute_operation(
    operation: RuntimeOperation,
    state: RuntimeState,
    env: RuntimeExecutionContext
)
    -> OperationResult
```

Acceptance:

- preconditions are evaluated
- operation effect executes
- input state is not mutated
- output state version increments
- provenance is appended
- emitted events are returned

---

## 4.15 handler_dispatcher

Responsibility:

Dispatch runtime events to Handlers.

Interface:

```text
dispatch_handlers(
    event: RuntimeEvent,
    state: RuntimeState,
    env: RuntimeExecutionContext
)
    -> HandlerDispatchResult
```

Acceptance:

- event matching works
- handler guards execute
- fallback operations execute when defined
- priority ordering is honored
- multiple handlers chain state transitions

---

## 4.16 migration_engine

Responsibility:

Execute Migration<A,B>.

Interface:

```text
execute_migration(
    migration: RuntimeMigration,
    source: RuntimeModel,
    env: RuntimeExecutionContext
)
    -> MigrationResult
```

Acceptance:

- source type is validated
- preserved invariants are captured
- mappings are applied
- transforms are applied
- deprecated properties are marked
- target model is constructed
- preserved invariants are revalidated
- failure produces INVALID semantics

---

## 4.17 variant_engine

Responsibility:

Select and apply Variants.

Interface:

```text
select_variants(
    variants: List<RuntimeVariant>,
    policy: VariantSelectionPolicy,
    env: RuntimeExecutionContext
)
    -> List<RuntimeVariant>

apply_variant(
    variant: RuntimeVariant,
    target: RuntimeValue,
    env: RuntimeExecutionContext
)
    -> RuntimeValue
```

Acceptance:

- NONE
- FIRST_MATCH
- ALL_MATCHES
- EXPLICIT

selection policies are implemented.

Applied variants preserve outer type and append provenance.

---

## 4.18 integrity_engine

Responsibility:

Evaluate Integrity<T>.

Interface:

```text
check_integrity(
    rule: RuntimeIntegrity,
    target: RuntimeValue,
    env: RuntimeExecutionContext
)
    -> IntegrityResult
```

Acceptance:

- verifier returns Bool/TriBool-compatible result
- failure emits integrity_failed
- repair handler may execute
- exactly one automatic recheck is supported
- unresolved repair does not loop indefinitely

---

## 4.19 completion_engine

Responsibility:

Evaluate Completion<T>.

Interface:

```text
evaluate_completion(
    completion: RuntimeCompletion,
    target: RuntimeValue,
    env: RuntimeExecutionContext
)
    -> CompletionResult
```

Acceptance:

- predicate false => continue
- predicate true => COMPLETE
- terminating operation executes when defined
- terminal state is applied when defined
- both may be used in defined order

---

## 4.20 execution_engine

Responsibility:

Coordinate full ScriptureUnit execution.

Interface:

```text
execute_unit(
    unit: TypedScriptureUnit,
    env: RuntimeEnvironment,
    input?: ExecutionInput
)
    -> ExecutionResult
```

Required execution order:

```text
1. load initial state/context/actors
2. enforce scope
3. apply mediator
4. evaluate constraints
5. execute primary operation
6. dispatch handlers
7. execute migration
8. select/apply variants
9. validate integrity
10. evaluate completion
11. return result + trace
```

Acceptance:

- order is preserved
- status precedence is preserved
- prior states remain available
- runtime failures return structured diagnostics

---

## 4.21 trace_recorder

Responsibility:

Record execution events.

Interface:

```text
record_trace(
    event: TraceEvent
) -> void
```

Acceptance:

Required event families exist:

```text
UNIT_START
SCOPE_CHECK
MEDIATOR_APPLY
CONSTRAINT_CHECK
OPERATION_START
OPERATION_END
HANDLER_DISPATCH
HANDLER_COMPLETE
MIGRATION_APPLY
VARIANT_SELECT
VARIANT_APPLY
INTEGRITY_CHECK
COMPLETION_CHECK
STATE_TRANSITION
UNIT_END
RUNTIME_ERROR
```

---

## 4.22 provenance_engine

Responsibility:

Preserve source-to-runtime lineage.

Interface:

```text
attach_provenance(
    runtime_object,
    source_symbol,
    source_library,
    source_span,
    source_version
)
```

Acceptance:

Every runtime object can answer:

```text
where did I come from?
```

Every resulting RuntimeState can identify the transitions that produced it.

---

## 4.23 diagnostics

Responsibility:

Provide structured diagnostics across all modules.

Interface:

```text
Diagnostic {
    code
    message
    severity
    span
    expected?
    actual?
    related_symbols?
}
```

Acceptance:

- modules do not depend on free-form strings as their only error surface
- diagnostic codes are stable
- source locations are preserved when available

---

# 5. Required Public Runtime Interfaces

At minimum, an implementation-complete runtime exposes:

```text
load_start_library(...)
build_runtime_environment(...)
execute_unit(...)
eval_expr(...)
execute_operation(...)
dispatch_handlers(...)
execute_migration(...)
select_variants(...)
apply_variant(...)
check_integrity(...)
evaluate_completion(...)
```

These may be methods or module functions, but equivalent capability must exist.

---

# 6. Required Runtime Data Interfaces

The implementation must provide concrete representations for:

```text
ParsedStartLibrary
ResolvedStartGraph
BoundStartGraph
SchemaValidStartGraph
DependencyValidStartGraph
TypeResolvedStartGraph
TypedStartGraph

RuntimeEnvironment
RuntimeState
RuntimeActor
RuntimeContext
RuntimeScope
RuntimeConstraint
RuntimeMediator
RuntimeOperation
RuntimeHandler
RuntimeMigration
RuntimeIntegrity
RuntimeCompletion
RuntimeVariant

ExecutionInput
ExecutionResult
ExecutionTrace
TraceEvent
RuntimeDiagnostic
```

---

# 7. Stub Rule

A required module is not complete if its required path contains:

```text
TODO
pass
NotImplemented
throw Unsupported
placeholder return
hard-coded success
hard-coded single fixture behavior
```

unless the code path is explicitly outside Core 0.1 scope.

---

# 8. Build Gate

Implementation completeness requires:

```text
B1. project builds successfully
B2. runtime package imports/loads successfully
B3. public interfaces resolve
B4. no missing required module
B5. no required module contains a blocking stub
```

---

# 9. Minimal Functional Acceptance

The implementation must successfully support this minimal flow:

```text
Start source
   ↓
load_start_library
   ↓
RuntimeEnvironment
   ↓
execute_unit
   ↓
RuntimeState S1
   ↓
ExecutionResult
```

Minimal scenario:

```text
Context C0
Actor A0
State S0
Scope P0
Constraint K0
Operation O0
Integrity I0
Completion Q0
```

Expected result:

```text
S0 --O0--> S1
I0(S1) == true
Q0(S1) == true
status == COMPLETE
```

---

# 10. Required Negative Acceptance

Implementation completeness also requires executable handling for:

```text
N1. invalid Start syntax
N2. missing import
N3. namespace ambiguity
N4. schema violation
N5. missing dependency
N6. dependency kind mismatch
N7. generic type mismatch
N8. scope denial
N9. mediator denial
N10. blocked constraint
N11. operation failure
N12. handler failure
N13. migration invariant failure
N14. invalid variant application
N15. integrity failure
N16. unresolved completion predicate
N17. execution-cycle detection
```

These do not need full test-suite coverage to satisfy the implementation gate, but the code paths must exist and return structured results.

---

# 11. Acceptance Evidence

Implementation COMPLETE requires evidence for every required module.

Recommended evidence format:

```text
Module:
Interface:
Implementation path:
Stub-free:
Build status:
Minimal invocation:
Observed result:
```

Example:

```text
Module: operation_engine
Interface: execute_operation(...)
Implementation path: runtime/operation_engine.*
Stub-free: yes
Build status: pass
Minimal invocation: Obey(S0)
Observed result: S1 version=1
```

---

# 12. Implementation Completeness Checklist

```text
[ ] start_parser implemented
[ ] import_resolver implemented
[ ] namespace_binder implemented
[ ] schema_validator implemented
[ ] dependency_checker implemented
[ ] type_resolver implemented
[ ] type_checker implemented
[ ] runtime_environment_builder implemented
[ ] runtime_state_factory implemented
[ ] expression_evaluator implemented
[ ] scope_engine implemented
[ ] mediator_engine implemented
[ ] constraint_engine implemented
[ ] operation_engine implemented
[ ] handler_dispatcher implemented
[ ] migration_engine implemented
[ ] variant_engine implemented
[ ] integrity_engine implemented
[ ] completion_engine implemented
[ ] execution_engine implemented
[ ] trace_recorder implemented
[ ] provenance_engine implemented
[ ] diagnostics implemented

[ ] required public runtime interfaces exposed
[ ] required runtime data interfaces implemented
[ ] project builds
[ ] runtime package loads
[ ] no required blocking stubs
[ ] minimal positive flow executes
[ ] required negative paths return structured diagnostics
```

---

# 13. Pass / Fail Rule

Implementation status is COMPLETE only if:

```text
all required module checkboxes == pass
AND
all required public interfaces exist
AND
build gate passes
AND
minimal positive flow passes
AND
required negative code paths exist
AND
no required blocking stubs remain
```

Otherwise:

```text
ImplementationStatus = PARTIAL
```

---

# 14. Boundary With Integration Completeness

Implementation completeness does not require proof that all modules work together across every end-to-end path.

That is the Integration Completeness gate.

Implementation COMPLETE means:

```text
all required runtime pieces exist
and each required interface is executable
```

Integration COMPLETE means:

```text
those pieces successfully compose across the full runtime path
```

This defines the concrete implementation-completeness gate for the Scripture as Compute runtime.
