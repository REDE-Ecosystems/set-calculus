# Scripture as Compute — Start Library Loading Pipeline

## 1. Purpose

This document defines the loading pipeline for Scripture as Compute Start libraries.

A Start library is never executed directly from source text.

It must pass through:

```text
source
  -> parse
  -> import resolution
  -> namespace binding
  -> schema validation
  -> dependency checks
  -> type resolution
  -> runtime conversion
```

Only a fully validated and typed Start library may become a RuntimeEnvironment.

---

# 2. Full Loading Pipeline

```text
.start source
   ↓
parseStart()
   ↓
ParsedStartLibrary
   ↓
resolveImports()
   ↓
ResolvedStartGraph
   ↓
bindNamespaces()
   ↓
BoundStartGraph
   ↓
validateSchema()
   ↓
SchemaValidStartGraph
   ↓
checkDependencies()
   ↓
DependencyValidStartGraph
   ↓
typeCheckStart()
   ↓
TypedStartGraph
   ↓
instantiateStart()
   ↓
RuntimeEnvironment
```

Each stage must either return a valid result or diagnostics.

No later stage may silently repair an earlier-stage error.

---

# 3. Loader Entry Point

```text
loadStartLibrary(
    source: StartSource,
    options?: StartLoadOptions
) -> StartLoadResult
```

```text
StartLoadOptions := {
    search_paths: List<Path>,
    registry_sources: List<LibraryRegistry>,
    allow_version_range: Bool,
    allow_optional_imports: Bool,
    namespace_policy: NamespacePolicy,
    dependency_policy: DependencyPolicy,
    provenance_mode: ProvenanceMode
}
```

Result:

```text
StartLoadResult :=
    Success<TypedStartGraph, RuntimeEnvironment>
  | Failure<StartLoadDiagnostics>
```

---

# 4. Stage 1 — Parsing

## 4.1 Input

```text
StartSource
```

Possible source forms:

```text
file path
string
stream
package artifact
embedded resource
```

---

## 4.2 Parse Result

```text
ParsedStartLibrary {
    metadata: ParsedStartMetadata
    imports: List<ParsedImportDecl>
    declarations: List<ParsedDeclaration>
    source_map: SourceMap
}
```

The parser validates syntax only.

It does not:

- resolve imports,
- validate symbol existence,
- type-check declarations,
- instantiate runtime objects.

---

## 4.3 Parse Errors

Minimum parse diagnostics:

```text
UnexpectedToken
UnexpectedEOF
InvalidStartMetadata
InvalidImportSyntax
InvalidDeclarationSyntax
DuplicateField
MissingRequiredField
InvalidLiteral
```

Example:

```text
error Start.Parse.MissingRequiredField
file: sinai.start
line: 4
field: namespace
```

---

# 5. Stage 2 — Import Resolution

Imports are resolved before namespace binding.

```text
resolveImports(root: ParsedStartLibrary)
    -> ResolvedStartGraph
```

---

## 5.1 Import Resolution Sources

Resolution order:

```text
1. explicit relative path
2. configured search paths
3. loaded library cache
4. configured registry
```

No network source is assumed unless explicitly configured.

---

## 5.2 Import Identity

Each imported library is identified by:

```text
LibraryIdentity {
    name
    namespace
    version
    schema_version
}
```

Two imports with the same namespace but incompatible identities are rejected unless one is explicitly aliased.

---

## 5.3 Version Resolution

Exact version:

```text
import "common-human.start" version "0.1.0"
```

Range form:

```text
import "common-human.start" version ">=0.1 <0.2"
```

If ranges are disabled:

```text
VersionRangeNotAllowed
```

If no compatible version is found:

```text
ImportVersionNotFound
```

---

## 5.4 Import Graph

Resolved imports form a graph:

```text
ResolvedStartGraph {
    root: ParsedStartLibrary
    libraries: Map<LibraryIdentity, ParsedStartLibrary>
    edges: Set<ImportEdge>
}
```

```text
ImportEdge {
    from
    to
    alias
    include
    exclude
    optional
}
```

---

# 6. Import Cycle Detection

Import cycles are detected before namespace binding.

Example:

```text
A.start -> B.start
B.start -> C.start
C.start -> A.start
```

Core 0.1 default:

```text
cyclic imports are invalid
```

Diagnostic:

```text
ImportCycleDetected
```

Future versions may permit explicit cycle-safe module groups.

---

# 7. Stage 3 — Namespace Binding

Namespace binding maps every declaration to a fully qualified symbol.

```text
bindNamespaces(graph)
    -> BoundStartGraph
```

Canonical symbol:

```text
namespace::identifier
```

Examples:

```text
sinai::Israel
sinai::TorahCompliance
human::Person
```

---

## 7.1 Namespace Table

```text
NamespaceTable {
    namespaces: Map<Identifier, NamespaceEntry>
}
```

```text
NamespaceEntry {
    identity: LibraryIdentity
    local_symbols: Map<Identifier, BoundSymbol>
    imports: Map<Identifier, ImportBinding>
}
```

---

## 7.2 Symbol Binding

Every declaration becomes:

```text
BoundSymbol {
    local_name
    qualified_name
    symbol_kind
    declaration
    library_identity
}
```

---

## 7.3 Reference Resolution Order

For an unqualified reference:

```text
Israel
```

resolution order is:

```text
1. current library local symbol
2. explicitly imported symbol
3. imported alias namespace
```

The loader does not guess between ambiguous imports.

Ambiguous example:

```text
A imports x::Israel
B imports y::Israel
reference = Israel
```

Diagnostic:

```text
AmbiguousReference
```

Required fix:

```text
x::Israel
```

or

```text
y::Israel
```

---

# 8. Namespace Collision Rules

Invalid:

```text
same namespace
same symbol name
different declaration
```

Diagnostic:

```text
DuplicateQualifiedSymbol
```

Valid through aliasing:

```text
import "a.start" as a
import "b.start" as b

a::State0
b::State0
```

---

# 9. Stage 4 — Schema Validation

Schema validation checks declaration structure.

```text
validateSchema(boundGraph)
    -> SchemaValidStartGraph
```

Schema validation is structural.

It does not yet resolve semantic compatibility.

---

# 10. Top-Level Schema Rules

Every Start library must contain:

```text
start metadata
schema_version
namespace
version
```

Declaration sections may be empty unless required by defaults.

If metadata contains:

```text
default_state
```

then a State declaration with that identifier must exist later during dependency validation.

---

# 11. Declaration Schema Validation

The loader validates the required fields for each declaration family.

---

## 11.1 State

Require:

```text
id
type = State
status
values
```

Optional:

```text
relations
provenance
extends
```

---

## 11.2 Actor

Require:

```text
id
type = Actor
role
```

Optional:

```text
capabilities
obligations
permissions
scopes
properties
```

---

## 11.3 Context

Require:

```text
id
type = Context
```

Context fields are optional.

---

## 11.4 Scope

Require:

```text
id
type = Scope
domain
```

---

## 11.5 Constraint

Require:

```text
id
type = Constraint
mode
predicate
```

---

## 11.6 Mediator

Require:

```text
id
type = Mediator
actor
protected_target
```

---

## 11.7 Operation

Require:

```text
id
type = Operation<I,O>
opcode
input_type
output_type
effect
```

---

## 11.8 Handler

Require:

```text
id
type = Handler<E>
trigger
operation
```

---

## 11.9 Migration

Require:

```text
id
type = Migration<A,B>
source_model
target_model
```

---

## 11.10 Integrity

Require:

```text
id
type = Integrity<T>
verifier
```

---

## 11.11 Completion

Require:

```text
id
type = Completion<T>
predicate
```

---

## 11.12 Variant

Require:

```text
id
type = Variant<T>
source
delta
```

---

# 12. Schema Version Validation

```text
schema_version
```

must be supported by the loader.

Example:

```text
schema_version: "0.1"
```

Unsupported:

```text
UnsupportedStartSchemaVersion
```

The loader must not silently reinterpret an unsupported schema.

---

# 13. Stage 5 — Dependency Checks

Dependency checks validate reference existence and declaration relationships.

```text
checkDependencies(schemaValidGraph)
    -> DependencyValidStartGraph
```

---

# 14. Dependency Graph

Every declaration may depend on others.

Example:

```text
Actor -> Constraint
Actor -> Scope
Constraint -> Operation
Mediator -> Actor
Mediator -> Operation
Handler -> Constraint
Handler -> Operation
Completion -> Operation
Completion -> State
Integrity -> Handler
Variant -> source declaration
Migration -> source/target model
```

Represented as:

```text
DeclarationDependencyGraph {
    nodes: Set<BoundSymbol>
    edges: Set<DependencyEdge>
}
```

---

# 15. Missing Dependency Rules

Example:

```text
actor Israel {
    obligations: { TorahCompliance }
}
```

If `TorahCompliance` is absent:

```text
MissingDependency(
    owner=sinai::Israel,
    dependency=sinai::TorahCompliance
)
```

---

# 16. Dependency Kind Validation

Reference existence is insufficient.

Example:

```text
constraint C0 {
    consequence: Israel
}
```

If `Israel` is Actor:

```text
InvalidDependencyKind(
    expected=Operation,
    actual=Actor
)
```

---

# 17. Declaration Dependency Cycles

Some declaration cycles are legal.

Example:

```text
Actor -> Scope
Scope -> Actor
```

This can be valid because both are declarative references.

Other cycles may be invalid when they require eager instantiation.

Core 0.1 classification:

```text
REFERENCE_CYCLE      allowed
INSTANTIATION_CYCLE  invalid
EXECUTION_CYCLE      handled at runtime
```

Diagnostic:

```text
InstantiationCycleDetected
```

---

# 18. Default Dependency Checks

If metadata declares:

```text
default_context
default_state
default_scope
```

the loader requires:

```text
default_context -> Context
default_state   -> State
default_scope   -> Scope
```

Kind mismatch is invalid.

---

# 19. Stage 6 — Start Type Checking

After dependencies are known:

```text
typeCheckStart(graph)
    -> TypedStartGraph
```

This uses the same semantic type system as the Scripture as Compute typed grammar.

---

# 20. Generic Type Resolution

Examples:

```text
Operation<State,State>
Handler<Event>
Migration<ModelOT,ModelNT>
Integrity<State>
Variant<TextArtifact>
Completion<State>
```

The loader resolves generic arguments before runtime instantiation.

---

# 21. Declaration Type Rules

## Operation

```text
effect input must unify with input_type
effect output must unify with output_type
```

---

## Handler

```text
trigger type must unify with Handler<E>
operation must be Operation<State,State>
fallback must be Operation<State,State>
```

---

## Migration

```text
source_model must unify with A
target_model must unify with B
```

---

## Integrity

```text
verifier must be T -> Bool
```

---

## Completion

```text
predicate must be T -> Bool
terminating_operation must be Operation<T,T>
terminal_state must be T
```

---

## Variant

```text
source must be T
semantic_effect must preserve T
```

---

# 22. Typed Start Graph

```text
TypedStartGraph {
    root: TypedStartLibrary
    libraries: Map<LibraryIdentity, TypedStartLibrary>
    namespaces: NamespaceTable
    dependencies: DeclarationDependencyGraph
    type_environment: TypeEnvironment
    provenance: ProvenanceGraph
}
```

Each declaration now contains resolved symbol references and semantic types.

---

# 23. Stage 7 — Runtime Conversion

```text
instantiateStart(typedGraph)
    -> RuntimeEnvironment
```

Runtime conversion is deterministic for a given typed library graph and instantiation input.

---

# 24. Runtime Conversion Order

Instantiate in dependency-safe order:

```text
1. contexts
2. actors
3. states
4. scopes
5. constraints
6. operations
7. mediators
8. handlers
9. migrations
10. integrity rules
11. completions
12. variants
```

This is an implementation order, not a semantic precedence rule.

---

# 25. Runtime Registry Construction

```text
RuntimeEnvironment {
    contexts
    actors
    states
    scopes
    constraints
    operations
    mediators
    handlers
    migrations
    integrity_rules
    completions
    variants
    provenance
}
```

Every runtime object retains:

```text
source_library
qualified_symbol
source_span
declaration_version
```

---

# 26. Runtime State Instantiation

The Start library State declaration is copied into runtime state.

```text
Start State Declaration
    ↓ instantiate
RuntimeState version=0
```

Example:

```text
sinai::CovenantInitial
```

becomes:

```text
RuntimeState {
    source = sinai::CovenantInitial
    version = 0
    ...
}
```

The source declaration remains immutable.

---

# 27. Runtime Actor Instantiation

Actor declarations become runtime actors.

```text
ActorDecl
    ↓
RuntimeActor
```

References to obligations and scopes are converted from symbols into runtime registry handles.

---

# 28. Runtime Operation Instantiation

Operations are compiled into executable runtime operations.

```text
Typed OperationDecl
    ↓
RuntimeOperation {
    opcode
    preconditions
    executable_effect
    emitted_events
}
```

The loader must not execute the effect during loading.

---

# 29. Runtime Handler Registration

Handlers are registered by trigger type.

```text
HandlerRegistry {
    EventType -> ordered handlers
}
```

Ordering:

```text
priority ascending/descending policy
then declaration order
```

Core 0.1 default:

```text
higher numeric priority first
then declaration order
```

---

# 30. Runtime Migration Registration

Migrations are indexed by:

```text
source type
target type
migration identifier
```

Example:

```text
MigrationRegistry[
    ModelOT,
    ModelNT,
    CovenantMigration
]
```

---

# 31. Runtime Variant Registry

Variants are indexed by target/source type and declaration order.

```text
VariantRegistry {
    target_type -> List<RuntimeVariant>
}
```

Selection policy remains runtime behavior.

---

# 32. Runtime Provenance Graph

Every loaded declaration contributes to a provenance graph.

```text
ProvenanceGraph {
    libraries
    imports
    declarations
    aliases
    instantiated_objects
}
```

This permits the runtime to answer:

```text
Where did this runtime object come from?
Which Start library defined it?
Which import path introduced it?
Which version was loaded?
```

---

# 33. Loader Caching

A validated typed library may be cached by:

```text
LibraryIdentity + content hash
```

Cache entry:

```text
TypedStartCacheEntry {
    identity
    content_hash
    typed_library
}
```

A changed content hash invalidates the cache.

---

# 34. Failure Atomicity

Start loading is atomic.

If any required stage fails:

```text
RuntimeEnvironment is not produced
```

No partial runtime registry may escape the loader.

Optional imports may be omitted only when explicitly marked optional.

---

# 35. Diagnostics

Minimum loader diagnostic families:

```text
Start.Parse.*
Start.Import.*
Start.Namespace.*
Start.Schema.*
Start.Dependency.*
Start.Type.*
Start.Instantiate.*
```

Examples:

```text
Start.Import.NotFound
Start.Import.CycleDetected
Start.Namespace.AmbiguousReference
Start.Namespace.DuplicateQualifiedSymbol
Start.Schema.UnsupportedVersion
Start.Dependency.Missing
Start.Dependency.InvalidKind
Start.Dependency.InstantiationCycle
Start.Type.GenericMismatch
Start.Instantiate.RuntimeConversionFailure
```

---

# 36. Minimal Loading Example

Root library:

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

import "common-human.start" as human

actor Israel : Actor = {
    role: community
    scopes: { IsraelScope }
}

state CovenantInitial : State = {
    status: RESOLVED
}
```

Pipeline:

```text
parse sinai.start
    ↓
resolve common-human.start
    ↓
bind:
    sinai::Israel
    sinai::CovenantInitial
    human::Person
    ↓
validate schema
    ↓
check dependency graph
    ↓
resolve types
    ↓
instantiate runtime
```

Result:

```text
RuntimeEnvironment {
    states: {
        sinai::CovenantInitial
    }

    actors: {
        sinai::Israel
    }

    ...
}
```

---

# 37. Loader Contract

Formal contract:

```text
loadStartLibrary(source)
    -> parse
    -> resolveImports
    -> bindNamespaces
    -> validateSchema
    -> checkDependencies
    -> typeCheckStart
    -> instantiateStart
    -> RuntimeEnvironment
```

Failure contract:

```text
any required stage failure
    -> diagnostics
    -> no RuntimeEnvironment
```

The loader therefore guarantees:

```text
RuntimeEnvironment
    implies
syntactically valid
+ imports resolved
+ namespaces bound
+ schema valid
+ dependencies valid
+ types resolved
```

This defines the Scripture as Compute Core 0.1 Start library loading pipeline.
