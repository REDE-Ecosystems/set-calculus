# Scripture as Compute — Abstract Syntax Tree and Parser

## 1. Purpose

This document defines the abstract syntax tree (AST) and parsing model for the Scripture as Compute typed grammar.

The parser consumes Scripture as Compute source text and produces a typed AST suitable for:

- static type checking,
- validation,
- interpretation,
- execution,
- migration analysis,
- provenance tracking,
- and later compilation.

Pipeline:

```text
source text
   ↓
lexer
   ↓
tokens
   ↓
parser
   ↓
AST
   ↓
type checker
   ↓
validated AST
   ↓
executor / analyzer
```

---

# 2. AST Root

Every parsed source file becomes a `ProgramNode`.

```text
ProgramNode := {
    declarations: List<DeclarationNode>,
    units: List<ScriptureUnitNode>
}
```

All top-level declarations derive from:

```text
DeclarationNode
```

---

# 3. Core AST Node Types

## 3.1 ProgramNode

```text
ProgramNode {
    declarations: List<DeclarationNode>
    units: List<ScriptureUnitNode>
}
```

---

## 3.2 StateDeclNode

```text
StateDeclNode {
    name: Identifier
    declared_type: State
    value: StateExprNode
}
```

---

## 3.3 ContextDeclNode

```text
ContextDeclNode {
    name: Identifier
    declared_type: Context
    value: ContextExprNode
}
```

---

## 3.4 ActorDeclNode

```text
ActorDeclNode {
    name: Identifier
    declared_type: Actor
    value: ActorExprNode
}
```

---

## 3.5 ConstraintDeclNode

```text
ConstraintDeclNode {
    name: Identifier
    declared_type: Constraint
    value: ConstraintExprNode
}
```

---

## 3.6 OperationDeclNode

```text
OperationDeclNode {
    name: Identifier
    input_type: TypeNode
    output_type: TypeNode
    value: OperationExprNode
}
```

Canonical declaration:

```text
operation O0 : Operation<State,State> = ...
```

---

## 3.7 HandlerDeclNode

```text
HandlerDeclNode {
    name: Identifier
    event_type: TypeNode
    value: HandlerExprNode
}
```

---

## 3.8 MediatorDeclNode

```text
MediatorDeclNode {
    name: Identifier
    declared_type: Mediator
    value: MediatorExprNode
}
```

---

## 3.9 ScopeDeclNode

```text
ScopeDeclNode {
    name: Identifier
    declared_type: Scope
    value: ScopeExprNode
}
```

---

## 3.10 MigrationDeclNode

```text
MigrationDeclNode {
    name: Identifier
    source_type: TypeNode
    target_type: TypeNode
    value: MigrationExprNode
}
```

---

## 3.11 IntegrityDeclNode

```text
IntegrityDeclNode {
    name: Identifier
    target_type: TypeNode
    value: IntegrityExprNode
}
```

---

## 3.12 VariantDeclNode

```text
VariantDeclNode {
    name: Identifier
    target_type: TypeNode
    value: VariantExprNode
}
```

---

## 3.13 CompletionDeclNode

```text
CompletionDeclNode {
    name: Identifier
    target_type: TypeNode
    value: CompletionExprNode
}
```

---

# 4. ScriptureUnit AST

```text
ScriptureUnitNode {
    name: Identifier

    context: RefNode
    actors: List<RefNode>
    initial_state: RefNode
    constraints: List<RefNode>
    operation: RefNode
    handlers: List<RefNode>
    mediator: Optional<RefNode>
    scope: RefNode
    migration: Optional<RefNode>
    integrity: Optional<RefNode>
    variants: List<RefNode>
    completion: Optional<RefNode>
}
```

The parser does not resolve references.

Example:

```text
operation: O0
```

becomes:

```text
RefNode("O0")
```

Reference binding occurs during semantic analysis.

---

# 5. Expression AST

All executable expressions derive from:

```text
ExprNode
```

Primary expression node types:

```text
LiteralNode
RefNode
FieldAccessNode
CallNode
CompareNode
LogicalNode
TransformNode
GuardedTransformNode
HandledTransformNode
MediatedTransformNode
ScopedExprNode
MigrationApplyNode
VariantApplyNode
IntegrityCheckNode
CompletionCheckNode
```

---

## 5.1 LiteralNode

```text
LiteralNode<T> {
    value: T
    literal_type: TypeNode
}
```

Examples:

```text
true
42
"Israel"
```

---

## 5.2 RefNode

```text
RefNode {
    name: Identifier
}
```

---

## 5.3 FieldAccessNode

```text
FieldAccessNode {
    target: ExprNode
    field: Identifier
}
```

Example:

```text
actor.role
```

AST:

```text
FieldAccess(
    target=Ref("actor"),
    field="role"
)
```

---

## 5.4 CallNode

```text
CallNode {
    callee: ExprNode
    arguments: List<ExprNode>
}
```

Example:

```text
I0(S1)
```

---

## 5.5 CompareNode

```text
CompareNode {
    left: ExprNode
    operator: CompareOperator
    right: ExprNode
}
```

```text
CompareOperator :=
    EQ
  | NE
  | LT
  | GT
  | LE
  | GE
  | IN
```

---

## 5.6 LogicalNode

```text
LogicalNode {
    operator: LogicalOperator
    operands: List<ExprNode>
}
```

```text
LogicalOperator :=
    AND
  | OR
  | NOT
```

---

# 6. Transform AST Nodes

## 6.1 TransformNode

Represents:

```text
S0 --O0--> S1
```

AST:

```text
TransformNode {
    source: ExprNode
    operation: RefNode
    target: ExprNode
}
```

---

## 6.2 GuardedTransformNode

Represents:

```text
S0 --[K0] O0--> S1
```

AST:

```text
GuardedTransformNode {
    source: ExprNode
    constraint: RefNode
    operation: RefNode
    target: ExprNode
}
```

---

## 6.3 HandledTransformNode

Represents:

```text
E0 -> H0 -> S1
```

AST:

```text
HandledTransformNode {
    event: ExprNode
    handler: RefNode
    target: ExprNode
}
```

---

## 6.4 MediatedTransformNode

Represents:

```text
A0 -> M0 -> O0 -> S1
```

AST:

```text
MediatedTransformNode {
    actor: RefNode
    mediator: RefNode
    operation: RefNode
    target: ExprNode
}
```

---

## 6.5 ScopedExprNode

Represents:

```text
P0 :: O0
```

AST:

```text
ScopedExprNode {
    scope: RefNode
    operation: RefNode
}
```

---

## 6.6 MigrationApplyNode

Represents:

```text
Model0 --G0--> Model1
```

AST:

```text
MigrationApplyNode {
    source: ExprNode
    migration: RefNode
    target: ExprNode
}
```

---

## 6.7 VariantApplyNode

Represents:

```text
T0 --V0--> T1
```

AST:

```text
VariantApplyNode {
    source: ExprNode
    variant: RefNode
    target: ExprNode
}
```

---

## 6.8 IntegrityCheckNode

Represents:

```text
I0(S1)
```

AST:

```text
IntegrityCheckNode {
    integrity: RefNode
    target: ExprNode
}
```

---

## 6.9 CompletionCheckNode

Represents:

```text
Q0(S1)
```

AST:

```text
CompletionCheckNode {
    completion: RefNode
    target: ExprNode
}
```

---

# 7. Type AST

Generic type expressions require their own nodes.

```text
TypeNode
NamedTypeNode
GenericTypeNode
```

## NamedTypeNode

```text
NamedTypeNode {
    name: Identifier
}
```

Examples:

```text
State
Actor
Context
```

## GenericTypeNode

```text
GenericTypeNode {
    base: Identifier
    arguments: List<TypeNode>
}
```

Examples:

```text
Operation<State,State>
Migration<Model0,Model1>
Integrity<State>
Variant<TextArtifact>
```

---

# 8. Lexer

The lexer converts characters into tokens.

Minimum token set:

```text
IDENTIFIER
STRING
INTEGER
BOOLEAN

COLON          :
COMMA          ,
DOT            .
EQUALS         =
LBRACE         {
RBRACE         }
LBRACKET       [
RBRACKET       ]
LPAREN         (
RPAREN         )
LANGLE         <
RANGLE         >

ARROW          ->
TRANSFORM_OPEN --[
TRANSFORM_MID  ]
TRANSFORM_END  -->
DOUBLE_COLON   ::

EQ             ==
NE             !=
LE             <=
GE             >=
LT             <
GT             >

KW_STATE
KW_CONTEXT
KW_ACTOR
KW_CONSTRAINT
KW_OPERATION
KW_HANDLER
KW_MEDIATOR
KW_SCOPE
KW_MIGRATION
KW_INTEGRITY
KW_VARIANT
KW_COMPLETION
KW_UNIT
KW_NONE
KW_AND
KW_OR
KW_NOT
KW_IN
```

Whitespace and comments are ignored outside string literals.

---

# 9. Parser Strategy

Core 0.1 uses a hand-written recursive-descent parser.

Top-level entry point:

```text
parseProgram()
```

Pseudo-code:

```text
parseProgram():
    program = ProgramNode()

    while not EOF:
        if next token starts declaration:
            program.declarations.append(parseDeclaration())
        else if next token == KW_UNIT:
            program.units.append(parseUnit())
        else:
            error("Expected declaration or unit")

    return program
```

---

# 10. Declaration Parsing

```text
parseDeclaration():
    switch peek():
        KW_STATE      -> parseStateDecl()
        KW_CONTEXT    -> parseContextDecl()
        KW_ACTOR      -> parseActorDecl()
        KW_CONSTRAINT -> parseConstraintDecl()
        KW_OPERATION  -> parseOperationDecl()
        KW_HANDLER    -> parseHandlerDecl()
        KW_MEDIATOR   -> parseMediatorDecl()
        KW_SCOPE      -> parseScopeDecl()
        KW_MIGRATION  -> parseMigrationDecl()
        KW_INTEGRITY  -> parseIntegrityDecl()
        KW_VARIANT    -> parseVariantDecl()
        KW_COMPLETION -> parseCompletionDecl()
        otherwise     -> error()
```

Generic declaration shape:

```text
keyword Identifier ":" Type "=" Expr
```

---

# 11. Type Parsing

```text
parseType():
    base = consume(IDENTIFIER or builtin type keyword)

    if match(LANGLE):
        args = []

        do:
            args.append(parseType())
        while match(COMMA)

        consume(RANGLE)

        return GenericTypeNode(base, args)

    return NamedTypeNode(base)
```

This parses:

```text
Operation<State,State>
Integrity<State>
Migration<Model0,Model1>
```

without hard-coding every generic combination.

---

# 12. ScriptureUnit Parsing

Source shape:

```text
unit U0 : ScriptureUnit = {
    context: C0,
    actors: [A0],
    initial_state: S0,
    constraints: [K0],
    operation: O0,
    handlers: [],
    mediator: none,
    scope: P0,
    migration: none,
    integrity: I0,
    variants: [],
    completion: Q0
}
```

Parser:

```text
parseUnit():
    consume(KW_UNIT)
    name = consume(IDENTIFIER)

    consume(COLON)
    consume("ScriptureUnit")
    consume(EQUALS)
    consume(LBRACE)

    fields = parseNamedFields()

    consume(RBRACE)

    return ScriptureUnitNode(
        name=name,
        context=fields["context"],
        actors=fields["actors"],
        initial_state=fields["initial_state"],
        constraints=fields["constraints"],
        operation=fields["operation"],
        handlers=fields["handlers"],
        mediator=fields["mediator"],
        scope=fields["scope"],
        migration=fields["migration"],
        integrity=fields["integrity"],
        variants=fields["variants"],
        completion=fields["completion"]
    )
```

Core 0.1 requires all ScriptureUnit fields to be present.

Optional values are represented explicitly with:

```text
none
```

---

# 13. Expression Parsing

Expression precedence:

```text
1. primary
2. field access / calls
3. comparison
4. not
5. and
6. or
7. transform expressions
```

Parser structure:

```text
parseExpr()
    -> parseTransform()

parseTransform()
    -> parseLogicalOr()
    -> optional transform suffix

parseLogicalOr()
    -> parseLogicalAnd()

parseLogicalAnd()
    -> parseUnary()

parseUnary()
    -> NOT parseUnary()
    -> parseComparison()

parseComparison()
    -> parsePostfix()
    -> optional comparator parsePostfix()

parsePostfix()
    -> parsePrimary()
    -> repeated field access or calls

parsePrimary()
    -> literal
    -> identifier
    -> parenthesized expression
```

---

# 14. Transform Parsing

## Basic transform

Input:

```text
S0 --O0--> S1
```

Parsing rule:

```text
StateRef "--" OperationRef "-->" StateRef
```

Produces:

```text
TransformNode
```

---

## Guarded transform

Input:

```text
S0 --[K0] O0--> S1
```

Parsing rule:

```text
StateRef "--[" ConstraintRef "]" OperationRef "-->" StateRef
```

Produces:

```text
GuardedTransformNode
```

---

## Mediated transform

Input:

```text
A0 -> M0 -> O0 -> S1
```

Because this shape is ambiguous at the token level, the parser initially produces:

```text
ChainNode {
    items: [A0, M0, O0, S1]
}
```

Semantic analysis resolves the chain into:

```text
MediatedTransformNode
```

after reference types are known.

This prevents the parser from guessing semantic types.

---

# 15. Parser / Type Checker Boundary

The parser verifies syntax.

The type checker verifies meaning.

The parser may accept:

```text
A0 -> K0 -> Q0 -> S1
```

as a syntactically valid chain.

The type checker rejects it if the resolved types do not match a valid transform form.

This separation is intentional:

```text
parser = structure
type checker = semantics
executor = behavior
```

---

# 16. Source Locations

Every AST node carries source metadata:

```text
SourceSpan {
    file: String
    start_line: Int
    start_column: Int
    end_line: Int
    end_column: Int
}
```

Base node:

```text
AstNode {
    span: SourceSpan
}
```

All AST nodes derive from `AstNode`.

This allows parser and type errors to point to exact source locations.

---

# 17. Parser Errors

Minimum parser error types:

```text
UnexpectedToken
ExpectedToken
InvalidDeclaration
InvalidTypeExpression
InvalidUnitField
DuplicateUnitField
MissingUnitField
UnterminatedExpression
UnexpectedEOF
```

Example:

```text
error: MissingUnitField
file: example.sac
line: 18
unit: U0
field: scope
```

---

# 18. Minimal Example Source

```text
context C0 : Context = C{
    covenantal: "Sinai"
}

actor A0 : Actor = A[Israel:community]

state S0 : State = S{
    status: RESOLVED
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
    state.status == COMPLETE
]

unit U0 : ScriptureUnit = {
    context: C0,
    actors: [A0],
    initial_state: S0,
    constraints: [K0],
    operation: O0,
    handlers: [],
    mediator: none,
    scope: P0,
    migration: none,
    integrity: I0,
    variants: [],
    completion: Q0
}
```

---

# 19. Minimal AST

The example parses approximately to:

```text
ProgramNode
|
+-- ContextDeclNode
|   +-- name: C0
|   +-- type: Context
|
+-- ActorDeclNode
|   +-- name: A0
|   +-- type: Actor
|
+-- StateDeclNode
|   +-- name: S0
|   +-- type: State
|
+-- ConstraintDeclNode
|   +-- name: K0
|   +-- CompareNode
|
+-- OperationDeclNode
|   +-- name: O0
|   +-- type: Operation<State,State>
|
+-- ScopeDeclNode
|   +-- name: P0
|
+-- IntegrityDeclNode
|   +-- name: I0
|
+-- CompletionDeclNode
|   +-- name: Q0
|
+-- ScriptureUnitNode
    +-- name: U0
    +-- context -> Ref(C0)
    +-- actors -> [Ref(A0)]
    +-- initial_state -> Ref(S0)
    +-- constraints -> [Ref(K0)]
    +-- operation -> Ref(O0)
    +-- handlers -> []
    +-- mediator -> none
    +-- scope -> Ref(P0)
    +-- migration -> none
    +-- integrity -> Ref(I0)
    +-- variants -> []
    +-- completion -> Ref(Q0)
```

---

# 20. Core 0.1 Parse Contract

For any syntactically valid source:

```text
parse(source) -> ProgramNode
```

For malformed source:

```text
parse(source) -> ParseError
```

Parsing does not require declarations to be semantically valid.

Semantic validation is a separate phase:

```text
typeCheck(parse(source)) -> TypedProgram | TypeError
```

The complete Core 0.1 front-end contract is therefore:

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

This defines the AST and parser baseline for Scripture as Compute Core 0.1.
