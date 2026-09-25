# Scripture as Compute — Runtime Test Completeness Gate

## 1. Purpose

This document defines the concrete test-completeness gate for the Scripture as Compute runtime.

The test gate answers one question:

> Is the defined runtime behavior repeatably demonstrated and protected across required positive paths, negative paths, state transitions, integrations, and failure modes?

Test completeness is distinct from:

```text
Architectural completeness
Implementation completeness
Integration completeness
```

A runtime may be architecturally, implementation, and integration complete while still failing the test-completeness gate.

---

# 2. Test Completeness Definition

Test completeness is COMPLETE only when:

```text
RequiredTestLayersPresent
AND
RequiredBehaviorCoveragePresent
AND
RequiredFailureCoveragePresent
AND
RequiredStatusCoveragePresent
AND
RequiredTraceAndProvenanceCoveragePresent
AND
RequiredRegressionCoveragePresent
AND
AllRequiredTestsPass
```

---

# 3. Required Test Layers

The runtime must include all of the following test layers:

```text
1. Unit tests
2. Module/component tests
3. Contract/interface tests
4. Integration tests
5. End-to-end tests
6. Negative/failure-path tests
7. Regression tests
```

These layers may share fixtures, but each layer must test a distinct responsibility.

---

# 4. Unit Tests

Unit tests validate isolated functions and rules.

Required unit-test targets:

```text
parser token handling
import path resolution
namespace lookup
schema field validation
dependency-kind validation
generic unification
TriBool logic
scope membership precedence
constraint mode evaluation
operation state immutability
handler ordering
migration mapping
variant policy selection
integrity verifier evaluation
completion predicate evaluation
status precedence
trace event construction
provenance entry construction
diagnostic construction
```

Acceptance:

```text
U1. Every pure rule or deterministic helper has at least one direct unit test.
U2. Every enum-driven branch has coverage.
U3. Three-valued logic truth tables are fully covered.
U4. State version increment logic is covered.
U5. No required helper remains test-free when behavior is nontrivial.
```

---

# 5. Module / Component Tests

Component tests validate each implemented runtime module through its public interface.

Required modules:

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

Acceptance for each module:

```text
C1. At least one valid-input test.
C2. At least one invalid-input or failure-path test.
C3. Public interface is exercised directly.
C4. Returned data type is asserted.
C5. Structured diagnostics are asserted where relevant.
```

---

# 6. Contract / Interface Tests

Contract tests verify module boundaries.

Required boundaries:

```text
parser -> import_resolver
import_resolver -> namespace_binder
namespace_binder -> schema_validator
schema_validator -> dependency_checker
dependency_checker -> type_resolver
type_resolver -> type_checker
type_checker -> runtime_environment_builder
runtime_environment_builder -> execution_engine

execution_engine -> scope_engine
execution_engine -> mediator_engine
execution_engine -> constraint_engine
execution_engine -> operation_engine
operation_engine -> handler_dispatcher
execution_engine -> migration_engine
execution_engine -> variant_engine
execution_engine -> integrity_engine
integrity_engine -> handler_dispatcher
execution_engine -> completion_engine
```

Acceptance:

```text
K1. Producer output is accepted directly by consumer.
K2. No test fixture manually reshapes data between modules.
K3. Qualified symbol identity is preserved across boundary.
K4. Semantic type information is preserved where required.
K5. Diagnostics preserve code and source span across boundary.
```

---

# 7. Integration Tests

Integration tests validate multiple modules acting together.

Required integration scenarios:

```text
I1. Start parse + import resolution
I2. imports + namespace binding
I3. binding + schema validation
I4. schema + dependency checking
I5. dependency graph + type resolution
I6. type resolution + type checking
I7. typed graph + runtime environment construction
I8. runtime environment + execution engine
I9. scope + mediator + constraint sequencing
I10. operation + handler event flow
I11. migration + live runtime state/model
I12. variant + integrity chain
I13. integrity failure + handler repair + recheck
I14. completion + execution termination
```

Acceptance:

```text
No manual bridge code is permitted inside the test.
```

---

# 8. End-to-End Tests

At least one full Start-to-result path must execute through the real runtime stack.

Required positive end-to-end case:

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
build RuntimeEnvironment
   ↓
load ScriptureUnit
   ↓
execute
   ↓
ExecutionResult
   ↓
ExecutionTrace + Provenance
```

Minimum scenario includes:

```text
Context
Actor
State
Scope
Constraint
Operation
Handler
Integrity
Completion
```

Expected:

```text
status == COMPLETE
```

---

# 9. Required Negative End-to-End Cases

At minimum, full-path tests must cover:

```text
E1. invalid Start syntax
E2. missing import
E3. import cycle
E4. namespace ambiguity
E5. schema violation
E6. missing dependency
E7. dependency kind mismatch
E8. generic type mismatch
E9. scope exclusion -> BLOCKED
E10. mediator denial -> BLOCKED
E11. constraint failure -> BLOCKED
E12. unresolved required value -> UNRESOLVED
E13. migration invariant failure -> INVALID
E14. invalid variant application -> INVALID or structured failure
E15. integrity failure without successful repair -> INVALID
E16. completion false -> RESOLVED
E17. completion true -> COMPLETE
E18. execution cycle detection
```

---

# 10. Runtime Status Coverage

All runtime statuses must be produced by automated tests:

```text
RESOLVED
UNRESOLVED
BLOCKED
INVALID
COMPLETE
```

Acceptance:

```text
S1. Each status is reached intentionally.
S2. Each status is asserted on ExecutionResult.
S3. Trace termination point is asserted.
S4. Diagnostic presence/absence is asserted as appropriate.
```

---

# 11. Constraint Mode Coverage

All Constraint modes require tests:

```text
REQUIRE
FORBID
ALLOW
LIMIT
CONDITION
```

Acceptance:

```text
Each mode has:
- one passing case
- one blocking/alternate case where meaningful
```

---

# 12. Variant Policy Coverage

All Core 0.1 policies require tests:

```text
NONE
FIRST_MATCH
ALL_MATCHES
EXPLICIT
```

Acceptance:

```text
V1. NONE applies no variant.
V2. FIRST_MATCH applies exactly first matching variant.
V3. ALL_MATCHES applies all matching variants in order.
V4. EXPLICIT applies only named variants.
V5. Applied variants preserve outer type.
V6. Provenance records applied variant IDs.
```

---

# 13. Handler Coverage

Handler tests must cover:

```text
event match
event non-match
guard pass
guard fail
fallback execution
priority ordering
multiple handlers
state chaining
synthetic runtime events
```

Acceptance:

```text
handler execution order == trace order
```

---

# 14. Mediator Coverage

Mediator tests must cover:

```text
actor permitted
actor denied
operation permitted
operation denied
mediator transform applied
protected target mismatch
```

Acceptance:

```text
denied operation is never executed
```

---

# 15. Migration Coverage

Migration tests must cover:

```text
valid source type
invalid source type
property replacement
property transform
deprecated property marking
preserved invariant success
preserved invariant failure
target model construction
```

Acceptance:

```text
migration trace identifies source migration declaration
```

---

# 16. Integrity Coverage

Integrity tests must cover:

```text
pass
fail
repair handler success
repair handler failure
single automatic recheck
no infinite repair loop
```

Acceptance:

```text
repair path uses normal handler dispatcher
```

---

# 17. Completion Coverage

Completion tests must cover:

```text
predicate false
predicate true
terminating operation only
terminal state only
terminating operation + terminal state
```

Acceptance:

```text
COMPLETE ends unit execution
```

---

# 18. Import and Namespace Coverage

Required tests:

```text
local symbol
qualified symbol
import alias
include filter
exclude filter
duplicate namespace
duplicate qualified symbol
ambiguous unqualified reference
missing imported symbol
version mismatch
optional import
```

---

# 19. Dependency Coverage

Required tests:

```text
valid dependency
missing dependency
wrong dependency kind
legal reference cycle
illegal instantiation cycle
default context validation
default state validation
default scope validation
```

---

# 20. Type-System Coverage

Required tests:

```text
named type resolution
generic type resolution
generic arity failure
type variable substitution
type mismatch
Operation<I,O> validation
Handler<E> validation
Migration<A,B> validation
Integrity<T> validation
Variant<T> validation
Completion<T> validation
```

---

# 21. State Transition Coverage

Required assertions for every operation transition test:

```text
input state unchanged
output state != input identity
output version == input version + 1
provenance appended
trace contains STATE_TRANSITION
operation identity preserved
```

---

# 22. Trace Coverage

Required trace event types must be exercised:

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

Acceptance:

```text
T1. Every required trace event type appears in at least one automated test.
T2. Event ordering is asserted in end-to-end tests.
T3. State-before/state-after links are asserted where applicable.
```

---

# 23. Provenance Coverage

Tests must prove that runtime objects retain:

```text
source library
qualified symbol
source span
library version
declaration version
transition lineage
```

Acceptance:

```text
P1. Loaded declaration provenance survives runtime conversion.
P2. Operation provenance survives state transition.
P3. Variant provenance survives application.
P4. Migration provenance survives model transition.
P5. Handler provenance survives repair or event path.
P6. Final state can be traced back to contributing source declarations.
```

---

# 24. Diagnostic Coverage

Every diagnostic family must be exercised:

```text
Start.Parse.*
Start.Import.*
Start.Namespace.*
Start.Schema.*
Start.Dependency.*
Start.Type.*
Start.Instantiate.*
Runtime.*
```

Acceptance:

```text
D1. Diagnostic code asserted.
D2. Message present.
D3. Source span asserted when available.
D4. Expected/actual asserted when applicable.
D5. Related symbol asserted when applicable.
```

---

# 25. Coverage Requirements

Core 0.1 test completeness is based on behavioral coverage, not a single line-coverage percentage.

Required:

```text
100% required module coverage
100% required public interface coverage
100% runtime status coverage
100% required Constraint mode coverage
100% Variant policy coverage
100% required trace event family coverage
100% required diagnostic family coverage
100% required negative acceptance scenario coverage
```

Line/branch coverage may be recorded, but does not substitute for the behavioral matrix.

Recommended code coverage floor:

```text
line coverage >= 85%
branch coverage >= 80%
```

These percentages are supporting evidence, not the primary gate.

---

# 26. Regression Test Requirement

Every fixed runtime defect must produce a regression test.

Acceptance:

```text
R1. Bug is reproduced by failing test before fix where practical.
R2. Fix causes test to pass.
R3. Regression test remains in suite.
R4. Test identifies the original failure condition.
```

---

# 27. Determinism Requirement

Tests that use deterministic input must produce deterministic output.

Acceptance:

```text
same Start library
+ same ScriptureUnit
+ same ExecutionInput
+ same variant policy
= same ExecutionResult
+ same semantic trace order
```

Non-semantic timestamps or generated IDs may be normalized.

---

# 28. Test Isolation Requirement

Acceptance:

```text
tests do not depend on execution order
tests do not mutate shared Start fixtures
tests do not require external network access unless explicitly marked
tests do not rely on hidden global runtime state
```

---

# 29. Test Fixtures

Minimum reusable fixtures:

```text
minimal_valid.start
invalid_syntax.start
missing_import.start
namespace_collision.start
dependency_failure.start
type_failure.start
scope_blocked.start
mediator_blocked.start
migration_valid.start
migration_invalid.start
variant_selection.start
integrity_repair.start
completion_complete.start
completion_resolved.start
unresolved.start
cycle.start
```

Fixtures should remain small and purpose-specific.

---

# 30. Test Evidence

For each required scenario, record:

```text
Test ID:
Layer:
Target module/path:
Input fixture:
Expected status/result:
Expected trace:
Expected diagnostics:
Observed result:
Pass/Fail:
```

---

# 31. Test Completeness Checklist

```text
[ ] unit test layer exists
[ ] component test layer exists
[ ] contract test layer exists
[ ] integration test layer exists
[ ] end-to-end test layer exists
[ ] failure-path test layer exists
[ ] regression test layer exists

[ ] every required module has positive coverage
[ ] every required module has negative/failure coverage
[ ] every required public interface is exercised

[ ] RESOLVED covered
[ ] UNRESOLVED covered
[ ] BLOCKED covered
[ ] INVALID covered
[ ] COMPLETE covered

[ ] REQUIRE covered
[ ] FORBID covered
[ ] ALLOW covered
[ ] LIMIT covered
[ ] CONDITION covered

[ ] NONE variant policy covered
[ ] FIRST_MATCH covered
[ ] ALL_MATCHES covered
[ ] EXPLICIT covered

[ ] handler dispatch coverage complete
[ ] mediator coverage complete
[ ] migration coverage complete
[ ] integrity coverage complete
[ ] completion coverage complete
[ ] import/namespace coverage complete
[ ] dependency coverage complete
[ ] type-system coverage complete
[ ] state-transition assertions complete
[ ] trace event family coverage complete
[ ] provenance coverage complete
[ ] diagnostic family coverage complete

[ ] minimal positive end-to-end flow passes
[ ] required negative end-to-end flows pass
[ ] imported-library end-to-end flow passes
[ ] integrity repair flow passes
[ ] migration flow passes
[ ] variant flow passes
[ ] unresolved flow passes
[ ] cycle-detection flow passes

[ ] regression rule enforced
[ ] deterministic-input rule passes
[ ] test isolation requirements pass

[ ] all required tests pass
```

---

# 32. Pass / Fail Rule

Test status is COMPLETE only if:

```text
all required test layers exist
AND
all required behavioral coverage exists
AND
all required positive scenarios pass
AND
all required negative scenarios pass
AND
all five runtime statuses are covered
AND
trace/provenance/diagnostic coverage requirements are met
AND
all required tests pass
```

Otherwise:

```text
TestStatus = PARTIAL
```

---

# 33. Boundary With Full Runtime Completeness

The runtime is fully complete only when:

```text
Architecture   == COMPLETE
Implementation == COMPLETE
Integration    == COMPLETE
Tests          == COMPLETE
```

The test-completeness gate certifies only the fourth dimension.

---

# 34. Test Done Definition

The runtime is test-complete when:

> Every required runtime behavior, status, module interface, integration seam, failure mode, provenance path, and trace family is covered by repeatable automated tests, and all required tests pass.

This defines the concrete test-completeness gate for the Scripture as Compute runtime.
