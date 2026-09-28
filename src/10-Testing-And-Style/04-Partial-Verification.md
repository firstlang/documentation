# Partial Verification

> **This feature is provisional. Its purpose and broad behavior are known, but many details of its syntax, verification model, and implementation still require clarification.**

Partial Verification is the process of checking intentionally incomplete First code without requiring it to be a complete executable program. The system that performs this process is the Partial Verifier.

Partial Verification is intended for code embedded in documentation, anchor semantic islands, and usage examples. These fragments should remain useful when their surrounding setup or irrelevant details have been omitted.

Partial Verification is not ordinary compilation. It uses the language parser, name resolution, and type system to verify the information that is present while representing omitted information explicitly.

## Principles

- A partial fragment uses ordinary First syntax wherever practical.
- Missing information does not automatically make a partial fragment invalid.
- Information present in the fragment is still checked.
- Omissions must not hide contradictions in the information that was provided.
- Removed, renamed, inaccessible, or incorrectly used APIs should be discoverable.
- Partial Verification produces analysis and notices. It does not introduce compiler errors.
- The Partial Verifier must distinguish an intentional omission from an accidental unresolved reference wherever possible.

Partial Verification may accept missing setup, omitted arguments, implied values, incomplete control flow, or other information that is unnecessary to demonstrate the point of an example. It should still discover invalid syntax, nonexistent members, impossible types, invalid visibility, unsatisfied generic constraints, and obsolete usage patterns when the supplied information is sufficient to do so.

## Anchor Semantic Islands

Anchors may contain brace-delimited semantic islands. A semantic island is either a partial-code island checked by the Partial Verifier or a resource island preserved for an implementation that understands its scheme.

Partial-code islands use single braces:

```first
{SomeClass}
{Files.readFileSync}
{calculateThings(...)}
```

Partial-code islands use ordinary lexical visibility and name-resolution rules. A reference that is not visible from its containing scope remains representable but produces a notice.

In the token-based editor, a resolved reference may retain the identity of its target rather than only its textual spelling. Renaming the target can then update the displayed reference. When raw text is parsed without identity information, the reference is resolved again using ordinary language rules.

First does not support function overloading. Functions use optional arguments, rest arguments, and sum types when they need to accept different call shapes. Consequently, `{calculateThings(...)}` does not refer to an overload family. It refers to `calculateThings` while deliberately omitting the details of its arguments.

Resource islands use the form `{scheme:payload}`. The scheme begins with an ASCII letter and may continue with ASCII letters, digits, `+`, `.`, or `-`. The payload is opaque to the language and follows URL-style UTF-8 percent encoding; literal `{` and `}` must be encoded as `%7B` and `%7D` so they cannot terminate the island. The language does not require any specific scheme to be supported by every implementation.

Resource islands are not checked as First code. Unknown schemes remain valid islands and may be rendered or ignored by editors according to their capabilities. Parsing a resource island must not require fetching, opening, or executing the referenced resource.

## Holes

The `...` token may represent deliberately omitted information in a context processed by the Partial Verifier:

```first
result = calculateThings(...)
```

In this example, the arguments are known to exist but are irrelevant to the example. The Partial Verifier determines whether some valid completion of the omitted argument list can satisfy the function signature.

This use of `...` is contextual. First also uses `...value` as the spread operator in ordinary code. A bare `...` used as a partial hole must only be accepted in a context where Partial Verification is active.

Holes do not mean that all surrounding constraints are discarded. The Partial Verifier should infer and retain every constraint it can obtain from the enclosing expression and the referenced declaration.

A Partial Verification hole is distinct from compiler `unknowable` recovery. A hole deliberately stands for omitted information that may have a valid completion. Unknowable instead propagates from an operation the ordinary compiler cannot currently interpret and short-circuits its containing statement.

## Implied Example Values

Usage examples often refer to values supplied by an implied surrounding environment:

```first
result = calculateThings(request)
```

The value `request` may be an example-local input rather than a declaration that exists in the program. The Partial Verifier may infer constraints on such a value from its use.

An assumed example-local value must remain distinguishable from a misspelled or inaccessible program symbol. The editor must surface that the value has been assumed rather than silently treating every unresolved identifier as valid.

The exact syntax or editor interaction used to confirm an assumed example-local value has not been decided.

## Relationship To Anchor Evaluation

Partial Verification checks incomplete language fragments contained in anchor semantic islands or referenced by anchors. It does not by itself determine whether a natural-language anchor is true of an implementation.

Anchor evaluation is a separate, continually running background process. Evaluation work is queued and processed when resources are available. That process may use the Partial Verifier when an anchor contains code, a semantic reference, or an incomplete usage example.

## Open Questions

- In which exact syntactic positions may a bare `...` hole appear?
- Can a hole omit only arguments, or can it stand for expressions, statements, types, declarations, or complete regions?
- How are the inferred types and constraints of holes represented?
- How does an author explicitly distinguish an assumed example-local value from an intended program reference?
- Which checks are mandatory for every partial fragment, and which checks depend on the amount of available context?
- What result states should the Partial Verifier expose?
- How should verification evidence and notices be represented?
- When is a partial result considered stale after referenced code changes?
- How much surrounding code may the Partial Verifier retrieve while checking a fragment?
- How are partial fragments lowered or isolated so that they never affect executable program output?
- Can controlled natural-language terms supply additional deterministic constraints to the Partial Verifier?
