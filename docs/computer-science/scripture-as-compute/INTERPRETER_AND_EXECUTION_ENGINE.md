# Scripture as Compute — Interpreter and Execution Engine

## 1. Purpose

This document defines the interpreter and execution engine for the Scripture as Compute typed AST.

The interpreter consumes a `TypedProgram` and produces runtime results by executing ScriptureUnit instances against runtime state.

Pipeline:

```text
TypedProgram
   ↓
RuntimeEnvironment
   ↓
execute(ScriptureUnit)
   ↓
scope enforcement
   ↓
mediator application
   ↓
constraint evaluation
   ↓
operation execution
   ↓
handler dispatch
   ↓
migration / variant application
   ↓
integrity validation
   ↓
completion evaluation
   ↓
ExecutionResult + ExecutionTrace
```

---

# 2. Runtime Model

## 2.1 RuntimeEnvironment

```text
RuntimeEnvironment {
    symbols: RuntimeSymbolTable
    states: Map<Identifier, RuntimeState>
    contexts: Map<Identifier, RuntimeContext>
    actors: Map<Identifier, RuntimeActor>
    scopes: Map<Identifier, RuntimeScope>
    handlers: HandlerRegistry
    migrations: MigrationRegistry
    variants: VariantRegistry
    trace: ExecutionTrace
}
```

The runtime environment is initialized from the `TypedProgram`.

---

## 2.2 RuntimeState

```text
RuntimeState {
    id: Identifier
    values: Map<Identifier, RuntimeValue>
    relations: Set<RuntimeRelation>
    status: StateStatus
    provenance: List<RuntimeProvenanceEntry>
    version: Int
}
```

State transitions are immutable by default.

An Operation receives one RuntimeState and returns a new RuntimeState.

```text
S0 -> O -> S1
```

The engine increments:

```text
S1.version = S0.version + 1
```

---

## 2.3 RuntimeContext

```text
RuntimeContext {
    temporal
    linguistic
    cultural
    covenantal
    textual
    situational
}
```

Context is read-only during a single execution unless a Migration explicitly changes the execution model.

---

## 2.4 RuntimeActor

```text
RuntimeActor {
    id
    role
    capabilities
    obligations
    permissions
    scope_membership
}
```

---

# 3. Execution Result

```text
ExecutionResult {
    initial_state: RuntimeState
    final_state: RuntimeState
    status: StateStatus
    trace: ExecutionTrace
    applied_handlers: List<Identifier>
    applied_migrations: List<Identifier>
    applied_variants: List<Identifier>
    diagnostics: List<RuntimeDiagnostic>
}
```

Possible final statuses:

```text
RESOLVED
UNRESOLVED
BLOCKED
INVALID
COMPLETE
```

---

# 4. Execution Trace

Every meaningful runtime action emits a trace event.

```text
ExecutionTrace {
    events: List<TraceEvent>
}
```

```text
TraceEvent {
    sequence: Int
    kind: TraceKind
    source_span: SourceSpan?
    actor: Identifier?
    state_before: Identifier?
    state_after: Identifier?
    symbol: Identifier?
    detail: Map<Identifier, RuntimeValue>
}
```

Trace kinds:

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

# 5. Interpreter Entry Point

```text
execute(
    program: TypedProgram,
    unit: ScriptureUnit,
    input?: ExecutionInput
) -> ExecutionResult
```

ExecutionInput may override runtime values:

```text
ExecutionInput {
    context_override: RuntimeContext?
    actors_override: List<RuntimeActor>?
    initial_state_override: RuntimeState?
    event: RuntimeEvent?
    variant_policy: VariantSelectionPolicy?
}
```

---

# 6. Execution Algorithm

High-level interpreter loop:

```text
executeUnit(unit):

    env = buildRuntimeEnvironment()

    state = loadInitialState(unit.initial_state)
    context = loadContext(unit.context)
    actors = loadActors(unit.actors)

    trace UNIT_START

    enforceScope(unit.scope, actors)

    if unit.mediator exists:
        applyMediator(unit.mediator, actors, state)

    evaluateConstraints(unit.constraints, context, actors, state)

    state = executeOperation(
        unit.operation,
        context,
        actors,
        state
    )

    state = dispatchHandlers(
        unit.handlers,
        context,
        actors,
        state
    )

    if unit.migration exists:
        state = executeMigration(
            unit.migration,
            context,
            actors,
            state
        )

    state = applySelectedVariants(
        unit.variants,
        context,
        actors,
        state
    )

    if unit.integrity exists:
        validateIntegrity(unit.integrity, state)

    status = evaluateCompletion(
        unit.completion,
        state
    )

    trace UNIT_END

    return ExecutionResult(...)
```

---

# 7. Scope Enforcement

Scope is enforced before the primary Operation executes.

```text
enforceScope(scope, actors)
```

For each actor:

```text
scope.boundary(actor) -> Bool
```

If false:

```text
status = BLOCKED
```

Trace:

```text
SCOPE_CHECK {
    actor
    scope
    result
}
```

---

## 7.1 Membership Resolution

Scope resolution order:

```text
1. explicit exclusion
2. explicit membership
3. boundary predicate
```

Rules:

```text
if actor in exclusions:
    BLOCKED

else if actor in members:
    allowed

else:
    evaluate boundary(actor)
```

Explicit exclusion wins over explicit membership.

---

# 8. Mediator Application

Mediator processing occurs before protected Operations.

```text
applyMediator(
    mediator,
    actors,
    state
)
```

Mediator responsibilities:

```text
1. validate actor eligibility
2. evaluate grants
3. evaluate denials
4. determine allowed operation path
5. apply mediator transforms if defined
```

---

## 8.1 Mediator Rule

Given:

```text
A -> M -> O
```

execution succeeds only if:

```text
M permits A
M permits O
```

If denied:

```text
BLOCKED
```

If mediator includes a transform:

```text
state = mediator.transform(state)
```

before the protected Operation runs.

Trace:

```text
MEDIATOR_APPLY
```

---

# 9. Constraint Evaluation

Each Constraint is evaluated in declaration order.

```text
evaluateConstraint(
    constraint,
    context,
    actors,
    state
) -> Bool
```

Modes:

## REQUIRE

```text
predicate must be true
```

false -> BLOCKED

## FORBID

```text
predicate must be false
```

true -> BLOCKED

## ALLOW

```text
predicate true permits path
```

## LIMIT

```text
predicate constrains allowed execution domain
```

## CONDITION

```text
predicate selects conditional execution path
```

---

## 9.1 Constraint Consequence

If a blocking Constraint has a consequence Operation:

```text
constraint.consequence
```

the engine executes it before returning the blocked result.

```text
state = consequence(state)
status = BLOCKED
```

---

# 10. Operation Execution

Operation execution:

```text
executeOperation(
    operation,
    context,
    actors,
    state
) -> RuntimeState
```

Runtime contract:

```text
Operation<State,State>
```

Execution stages:

```text
1. verify preconditions
2. emit OPERATION_START
3. evaluate effect
4. construct new state
5. append provenance
6. increment state version
7. emit STATE_TRANSITION
8. emit OPERATION_END
```

---

## 10.1 State Immutability

The engine does not mutate `state` in place.

```text
S1 = clone(S0)
apply effects to S1
```

Then:

```text
S0 remains in trace history
```

---

# 11. Handler Dispatch

Handlers react to runtime events or conditions.

```text
dispatchHandlers(
    handlers,
    context,
    actors,
    state,
    event?
)
```

Dispatch order is declaration order unless explicit priority is added later.

---

## 11.1 Handler Matching

For each Handler:

```text
if trigger matches event:
    if guard exists:
        evaluate guard
    if guard passes:
        execute handler.operation
    else if fallback exists:
        execute fallback
```

---

## 11.2 Synthetic Runtime Events

The engine may emit built-in events:

```text
constraint_failed
scope_blocked
mediator_denied
operation_completed
integrity_failed
migration_completed
variant_applied
completion_reached
```

Handlers may subscribe to these event types.

---

## 11.3 Handler Result

A Handler Operation returns a State.

```text
state_before
  -> Handler.operation
  -> state_after
```

Multiple handlers chain:

```text
S0 -> H1 -> S1 -> H2 -> S2
```

---

# 12. Migration Execution

Migration changes the execution model or model-level state.

```text
executeMigration(
    migration,
    context,
    actors,
    state
)
```

Migration stages:

```text
1. validate source model
2. capture preserved invariants
3. apply replacements
4. apply transformations
5. mark deprecated properties
6. construct target model
7. validate preserved invariants
8. emit MIGRATION_APPLY
```

---

## 12.1 Migration Mapping

For:

```text
external_constraint => internalized_constraint
```

runtime execution:

```text
target[property_b] =
    transform(source[property_a])
```

If no transform is supplied:

```text
target[property_b] = source[property_a]
```

---

## 12.2 Migration Failure

Migration fails if:

```text
source model does not match
required preserved invariant fails
required target property cannot be produced
```

Result:

```text
INVALID
```

unless a migration failure Handler resolves it.

---

# 13. Variant Selection

Variants may exist simultaneously.

The interpreter must select which Variant applies.

```text
selectVariant(
    variants,
    context,
    policy
) -> List<Variant>
```

Core 0.1 selection policies:

```text
NONE
FIRST_MATCH
ALL_MATCHES
EXPLICIT
```

---

## 13.1 FIRST_MATCH

Evaluate Variants in declaration order.

Apply first Variant whose selection condition resolves true.

---

## 13.2 ALL_MATCHES

Apply every matching Variant sequentially.

```text
S0 -> V1 -> S1 -> V2 -> S2
```

---

## 13.3 EXPLICIT

ExecutionInput must provide:

```text
variant_ids
```

Only those Variants are applied.

---

## 13.4 Variant Application

```text
applyVariant(
    variant,
    target
)
```

A Variant preserves outer type.

For `Variant<State>`:

```text
State -> Variant -> State
```

For textual artifacts:

```text
TextArtifact -> Variant -> TextArtifact
```

Every applied Variant appends provenance.

---

# 14. Integrity Checks

Integrity executes after operation, migration, and variant application unless configured otherwise later.

```text
validateIntegrity(
    integrity,
    target
)
```

Contract:

```text
Integrity<T> : T -> Bool
```

If true:

```text
continue
```

If false:

```text
emit integrity_failed
dispatch matching handlers
```

If no Handler resolves the failure:

```text
status = INVALID
```

---

## 14.1 Integrity Recheck

If an integrity failure Handler modifies the State:

```text
S_bad -> H -> S_repaired
```

the engine re-runs the same Integrity check once.

```text
I(S_repaired)
```

Core 0.1 permits one automatic repair cycle to avoid infinite loops.

---

# 15. Completion Evaluation

Completion is evaluated last.

```text
evaluateCompletion(
    completion,
    state
)
```

If no Completion exists:

```text
status = RESOLVED
```

If Completion predicate is false:

```text
status = RESOLVED
```

If true:

```text
status = COMPLETE
```

---

## 15.1 Terminating Operation

If Completion defines:

```text
terminating_operation
```

then:

```text
if Q(state):
    state = terminating_operation(state)
    status = COMPLETE
```

---

## 15.2 Terminal State

If Completion defines:

```text
terminal_state
```

then after predicate success:

```text
state = terminal_state
status = COMPLETE
```

If both are defined:

```text
1. execute terminating_operation
2. resolve to terminal_state
```

---

# 16. Runtime Status Resolution

Status precedence:

```text
INVALID
BLOCKED
COMPLETE
UNRESOLVED
RESOLVED
```

Meaning:

- `INVALID` overrides all other states.
- `BLOCKED` indicates execution was prevented.
- `COMPLETE` indicates successful terminal resolution.
- `UNRESOLVED` indicates insufficient runtime information.
- `RESOLVED` indicates successful nonterminal execution.

---

# 17. Unresolved Runtime Values

Some expressions may depend on unavailable Context.

Example:

```text
context.temporal == unknown
```

If a required predicate cannot evaluate to Bool:

```text
UNRESOLVED
```

The runtime distinguishes:

```text
false
```

from:

```text
unknown
```

Core runtime logic therefore uses:

```text
TriBool :=
    TRUE
  | FALSE
  | UNKNOWN
```

---

# 18. Runtime Expression Evaluation

```text
eval(expr, env) -> RuntimeValue
```

Supported Core 0.1 expression forms:

```text
Literal
Ref
FieldAccess
Call
Compare
Logical
IntegrityCheck
CompletionCheck
```

Comparison with unknown values yields:

```text
UNKNOWN
```

Logical evaluation follows three-valued logic.

---

# 19. Three-Valued Logic

## AND

```text
TRUE    and TRUE    = TRUE
TRUE    and FALSE   = FALSE
TRUE    and UNKNOWN = UNKNOWN
FALSE   and *       = FALSE
UNKNOWN and UNKNOWN = UNKNOWN
```

## OR

```text
TRUE    or *        = TRUE
FALSE   or FALSE    = FALSE
FALSE   or UNKNOWN  = UNKNOWN
UNKNOWN or UNKNOWN  = UNKNOWN
```

## NOT

```text
not TRUE    = FALSE
not FALSE   = TRUE
not UNKNOWN = UNKNOWN
```

---

# 20. Runtime Diagnostics

Minimum RuntimeDiagnostic codes:

```text
ScopeBlocked
MediatorDenied
ConstraintBlocked
ConstraintUnresolved
OperationFailure
HandlerFailure
MigrationFailure
VariantSelectionFailure
VariantApplicationFailure
IntegrityFailure
CompletionFailure
UnresolvedRuntimeValue
InvalidRuntimeType
ExecutionCycleDetected
```

---

# 21. Cycle Protection

Handlers, migrations, or completion operations may create cycles.

The engine tracks:

```text
ExecutionFrame {
    symbol
    state_version
}
```

If the same frame repeats beyond a configured threshold:

```text
ExecutionCycleDetected
```

Core 0.1 default:

```text
max_same_frame_repetitions = 1
```

unless a Completion explicitly defines a repeated process.

---

# 22. Minimal Runtime Example

Source model:

```text
context C0 : Context = C{
    covenantal: "Sinai"
}

actor A0 : Actor = A[Israel:community]

state S0 : State = S{
    status: RESOLVED,
    obedient: false
}

constraint K0 : Constraint = K[
    actor.scope_membership in P0
]

operation O0 : Operation<State,State> = O(obey)

scope P0 : Scope = P{Israel}

integrity I0 : Integrity<State> = I[
    state.status != INVALID
]

completion Q0 : Completion<State> = Q[
    state.obedient == true
]
```

Execution:

```text
execute(U0)
```

Trace:

```text
UNIT_START

SCOPE_CHECK
A0 in P0
=> TRUE

CONSTRAINT_CHECK
K0
=> TRUE

OPERATION_START
O0

STATE_TRANSITION
S0.obedient=false
    ->
S1.obedient=true

OPERATION_END

INTEGRITY_CHECK
I0(S1)
=> TRUE

COMPLETION_CHECK
Q0(S1)
=> TRUE

UNIT_END
status=COMPLETE
```

Result:

```text
ExecutionResult {
    initial_state: S0,
    final_state: S1,
    status: COMPLETE
}
```

---

# 23. Minimal Mediated Example

```text
A0 -> M0 -> O0 -> S1
```

Runtime:

```text
checkScope(A0)
applyMediator(M0,A0)
evaluateConstraints()
execute(O0)
validateIntegrity()
evaluateCompletion()
```

If mediator denies:

```text
status = BLOCKED
```

---

# 24. Minimal Migration Example

```text
G0 : Migration<ModelOT,ModelNT>
```

Execution:

```text
ModelOT
   --G0-->
ModelNT
```

Trace:

```text
MIGRATION_APPLY
preserve semantic_contract
replace repeated_operation
transform mediation
expand scope
validate invariants
```

If preserved invariant fails:

```text
INVALID
```

---

# 25. Minimal Variant Example

```text
V1 : Variant<TextArtifact>
V2 : Variant<TextArtifact>
```

Policy:

```text
FIRST_MATCH
```

Execution:

```text
evaluate V1 condition
if true:
    apply V1
else:
    evaluate V2
```

Trace records:

```text
VARIANT_SELECT
VARIANT_APPLY
```

---

# 26. Core 0.1 Execution Contract

A valid execution follows:

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

Formal contract:

```text
interpret(
    typedProgram,
    scriptureUnit,
    executionInput
)
    -> ExecutionResult
```

The engine never discards prior states.

Every transition contributes to provenance:

```text
S0
 -> S1
 -> S2
 -> ...
```

with a trace explaining how each state was produced.

This defines the Scripture as Compute Core 0.1 interpreter and execution engine.
