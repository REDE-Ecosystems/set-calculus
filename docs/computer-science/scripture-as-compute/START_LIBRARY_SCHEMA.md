# Scripture as Compute — Start Library Schema

## 1. Purpose

The Start library is a loadable declaration package that provides the initial executable world for the Scripture as Compute runtime.

A Start library is not itself runtime state.

It is a source/provenance artifact that contains declarations from which runtime objects are instantiated.

```text
StartLibrary
    ↓ load
Declaration Registry
    ↓ instantiate
RuntimeEnvironment
    ↓ execute
RuntimeState transitions
```

The runtime must not mutate the Start library in place.

---

# 2. Top-Level Start Schema

```text
StartLibrary := {
    start: StartMetadata,
    imports: List<ImportDecl>,

    contexts: List<ContextDecl>,
    actors: List<ActorDecl>,
    states: List<StateDecl>,
    scopes: List<ScopeDecl>,
    constraints: List<ConstraintDecl>,
    mediators: List<MediatorDecl>,
    operations: List<OperationDecl>,
    handlers: List<HandlerDecl>,
    migrations: List<MigrationDecl>,
    integrity_rules: List<IntegrityDecl>,
    completions: List<CompletionDecl>,
    variants: List<VariantDecl>
}
```

All declaration identifiers must be unique within a Start library namespace.

---

# 3. Metadata

```text
StartMetadata := {
    name: Identifier,
    version: String,
    namespace: Identifier,
    description: String?,
    provenance: ProvenanceRef?,
    schema_version: String,
    default_context: Identifier?,
    default_state: Identifier?,
    default_scope: Identifier?
}
```

Example:

```text
start {
    name: "Sinai"
    version: "0.1.0"
    namespace: "sinai"
    schema_version: "0.1"
    default_context: SinaiContext
    default_state: CovenantInitial
    default_scope: Israel
}
```

---

# 4. Imports

A Start library may depend on other Start libraries.

```text
ImportDecl := {
    library: String,
    version: String?,
    alias: Identifier?,
    include: List<Identifier>?,
    exclude: List<Identifier>?
}
```

Example:

```text
import "common-human.start" as human
```

Qualified references:

```text
human::Person
human::AliveState
```

Import resolution must preserve source provenance.

---

# 5. Context Declarations

```text
ContextDecl := {
    id: Identifier,
    type: "Context",
    temporal: Value?,
    linguistic: Value?,
    cultural: Value?,
    covenantal: Value?,
    textual: Value?,
    situational: Map<Identifier, Value>,
    extends: Identifier?
}
```

Example:

```text
context SinaiContext : Context = {
    temporal: "Sinai"
    linguistic: "Hebrew"
    covenantal: "Sinai"
}
```

---

# 6. Actor Declarations

```text
ActorDecl := {
    id: Identifier,
    type: "Actor",
    role: Symbol,
    capabilities: Set<Symbol>,
    obligations: Set<Identifier>,
    permissions: Set<Symbol>,
    scopes: Set<Identifier>,
    properties: Map<Identifier, Value>
}
```

Example:

```text
actor Israel : Actor = {
    role: community
    capabilities: { obey, violate, repent }
    obligations: { TorahCompliance }
    scopes: { IsraelScope }
}
```

Actor obligations reference Constraint declarations.

---

# 7. State Declarations

```text
StateDecl := {
    id: Identifier,
    type: "State",
    values: Map<Identifier, Value>,
    relations: Set<RelationDecl>,
    status: StateStatus,
    provenance: ProvenanceRef?,
    extends: Identifier?
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

Example:

```text
state CovenantInitial : State = {
    status: RESOLVED
    values: {
        obedient: false
        covenant_active: true
    }
}
```

---

# 8. Scope Declarations

```text
ScopeDecl := {
    id: Identifier,
    type: "Scope",
    domain: Symbol,
    members: Set<Identifier>,
    exclusions: Set<Identifier>,
    boundary: PredicateExpr?
}
```

Example:

```text
scope IsraelScope : Scope = {
    domain: Israel
    members: { Israel }
    exclusions: {}
}
```

Members and exclusions reference Actor declarations.

---

# 9. Constraint Declarations

```text
ConstraintDecl := {
    id: Identifier,
    type: "Constraint",
    mode: ConstraintMode,
    predicate: PredicateExpr,
    target: TargetRef?,
    consequence: Identifier?,
    description: String?
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

Example:

```text
constraint TorahCompliance : Constraint = {
    mode: REQUIRE
    predicate: actor.scope_membership in IsraelScope
}
```

If `consequence` is present, it references an Operation declaration.

---

# 10. Mediator Declarations

```text
MediatorDecl := {
    id: Identifier,
    type: "Mediator",
    actor: Identifier,
    grants: Set<Symbol>,
    denies: Set<Symbol>,
    transforms: Set<Identifier>,
    protected_target: TargetRef
}
```

Example:

```text
mediator AaronicPriesthood : Mediator = {
    actor: Priest
    grants: { sanctuary_access }
    denies: {}
    transforms: { PresentOffering }
    protected_target: Sanctuary
}
```

Mediator transforms reference Operation declarations.

---

# 11. Operation Declarations

```text
OperationDecl := {
    id: Identifier,
    type: OperationType,
    opcode: Symbol,
    input_type: TypeRef,
    output_type: TypeRef,
    preconditions: List<Identifier>,
    effect: EffectExpr,
    emits: List<RuntimeEventDecl>,
    description: String?
}
```

Canonical type:

```text
Operation<State,State>
```

Example:

```text
operation Obey : Operation<State,State> = {
    opcode: obey
    input_type: State
    output_type: State
    preconditions: { TorahCompliance }
    effect: {
        state.obedient = true
    }
}
```

---

# 12. Handler Declarations

```text
HandlerDecl := {
    id: Identifier,
    type: HandlerType,
    trigger: EventPattern,
    guard: Identifier?,
    operation: Identifier,
    fallback: Identifier?,
    priority: Int?
}
```

Example:

```text
handler ViolationHandler : Handler<Event> = {
    trigger: violation
    operation: ApplyConsequence
}
```

Guard references a Constraint.

Operation and fallback reference Operations.

---

# 13. Migration Declarations

```text
MigrationDecl := {
    id: Identifier,
    type: MigrationType,
    source_model: TypeRef,
    target_model: TypeRef,
    preserved: Set<InvariantExpr>,
    replaced: List<MigrationMapping>,
    transformed: List<MigrationMapping>,
    deprecated: Set<Identifier>
}
```

```text
MigrationMapping := {
    from: PropertyRef,
    to: PropertyRef,
    transform: Identifier?
}
```

Example:

```text
migration CovenantMigration : Migration<ModelOT,ModelNT> = {
    preserved: {
        covenant_identity
    }

    transformed: {
        external_constraint => internalized_constraint
        repeated_operation => completed_operation
        mediated_access => direct_access
        restricted_scope => expanded_scope
    }
}
```

---

# 14. Integrity Rule Declarations

```text
IntegrityDecl := {
    id: Identifier,
    type: IntegrityType,
    invariant: InvariantExpr,
    verifier: PredicateExpr,
    failure_condition: PredicateExpr?,
    repair_handler: Identifier?,
    provenance_rule: PredicateExpr?
}
```

Example:

```text
integrity ValidState : Integrity<State> = {
    invariant: state.status != INVALID
    verifier: state.status != INVALID
}
```

If `repair_handler` exists, it references a Handler.

---

# 15. Completion Declarations

```text
CompletionDecl := {
    id: Identifier,
    type: CompletionType,
    predicate: PredicateExpr,
    terminating_operation: Identifier?,
    terminal_state: Identifier?
}
```

Example:

```text
completion ObedienceReached : Completion<State> = {
    predicate: state.obedient == true
}
```

The terminating operation references an Operation.

The terminal state references a State.

---

# 16. Variant Declarations

```text
VariantDecl := {
    id: Identifier,
    type: VariantType,
    source: Identifier,
    condition: PredicateExpr?,
    delta: DeltaExpr,
    cause: Symbol?,
    semantic_effect: Identifier?,
    provenance: ProvenanceRef?
}
```

Example:

```text
variant LXXReading : Variant<TextArtifact> = {
    source: BaseText
    condition: context.linguistic == "Greek"
    delta: ...
    cause: translation
}
```

If `semantic_effect` exists, it references a type-compatible Operation or Transform.

---

# 17. Reference Rules

References may be:

```text
local
qualified
imported
```

Examples:

```text
Israel
sinai::Israel
human::Person
```

Resolution order:

```text
1. local declaration
2. explicit import alias
3. library namespace
```

Ambiguous references are invalid.

---

# 18. Start Namespace Rules

Every Start library exposes a namespace.

```text
namespace::symbol
```

Example:

```text
sinai::CovenantInitial
sinai::Israel
sinai::TorahCompliance
```

The runtime may mount multiple libraries simultaneously:

```text
load("eden.start")
load("sinai.start")
load("gospel.start")
```

Mounted libraries must not overwrite each other's symbols.

---

# 19. Instantiation

Loading a Start library has two stages.

## Stage 1: declaration load

```text
StartLibrary
    ↓
DeclarationRegistry
```

No runtime mutation occurs here.

## Stage 2: instantiation

```text
instantiate(start)
```

creates:

```text
RuntimeContext
RuntimeActor
RuntimeState
RuntimeScope
RuntimeConstraint
RuntimeMediator
RuntimeOperation
RuntimeHandler
RuntimeMigration
RuntimeIntegrity
RuntimeCompletion
RuntimeVariant
```

---

# 20. Default Instantiation

If metadata supplies defaults:

```text
default_context
default_state
default_scope
```

then:

```text
runtime = instantiate(start)
```

uses those declarations automatically.

Otherwise, the caller must specify them.

---

# 21. Start Load Contract

```text
loadStart(source)
    -> ParsedStartLibrary
```

Then:

```text
bindStart(parsed)
    -> BoundStartLibrary
```

Then:

```text
typeCheckStart(bound)
    -> TypedStartLibrary
```

Then:

```text
instantiateStart(typed)
    -> RuntimeEnvironment
```

Full pipeline:

```text
.start source
   ↓ parse
ParsedStartLibrary
   ↓ bind
BoundStartLibrary
   ↓ type-check
TypedStartLibrary
   ↓ instantiate
RuntimeEnvironment
```

---

# 22. Minimal Start Example

```text
start {
    name: "Sinai"
    version: "0.1.0"
    namespace: "sinai"
    schema_version: "0.1"
    default_context: SinaiContext
    default_state: CovenantInitial
    default_scope: IsraelScope
}

context SinaiContext : Context = {
    covenantal: "Sinai"
    linguistic: "Hebrew"
}

actor Israel : Actor = {
    role: community
    capabilities: { obey, violate, repent }
    scopes: { IsraelScope }
}

state CovenantInitial : State = {
    status: RESOLVED
    values: {
        obedient: false
        covenant_active: true
    }
}

scope IsraelScope : Scope = {
    domain: Israel
    members: { Israel }
}

constraint TorahCompliance : Constraint = {
    mode: REQUIRE
    predicate: actor.scope_membership in IsraelScope
}

operation Obey : Operation<State,State> = {
    opcode: obey
    input_type: State
    output_type: State
    preconditions: { TorahCompliance }

    effect: {
        state.obedient = true
    }
}

integrity ValidState : Integrity<State> = {
    verifier: state.status != INVALID
}

completion ObedienceReached : Completion<State> = {
    predicate: state.obedient == true
}
```

Instantiation:

```text
start = load("sinai.start")

runtime = instantiate(start)

runtime.initial_state
    = sinai::CovenantInitial
```

---

# 23. Core Design Principle

The Start library defines:

```text
what exists when execution begins
```

The ScriptureUnit defines:

```text
what acts on that world
```

The runtime defines:

```text
how the transformation resolves
```

The execution trace defines:

```text
what actually happened
```

Formal separation:

```text
StartLibrary != RuntimeState
```

Instead:

```text
StartLibrary
    -> instantiate
RuntimeState S0
    -> execute
RuntimeState S1
```

This defines the Start Library Schema for Scripture as Compute Core 0.1.
