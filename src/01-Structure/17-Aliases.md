# Aliases

An alias gives a type expression a name. Aliases use `is alias of`.

```first
Password is alias of string
```

An alias is purely structural. It does not create a more-derived nominal type.

```first
Password is alias of string

Passphrase is string (
)
```

`Password` is another name for `string`. `Passphrase` is a distinct nominal type.

## Type Expressions

Simple aliases can name primitive, class, selection, generic, union, intersection, array, and callable types.

```first
Primitive is alias of string or int or boolean
NamedItems is alias of Item[]
Reader is alias of () is string
Transform is alias of (value is int) is string
```

Use aliases when an inline annotation would need precedence grouping or a mixed composition.

```first
LoadSource is alias of Cached or Remote
Combined is alias of A and B or C

load() is LoadSource[] (
)
```

Inline type annotations stay visually linear. Standalone grouping parentheses are not inline type grouping.

## Generic Aliases

Generic aliases declare type parameters in the alias header, after the alias name and before `is alias of`.

```first
ResultOf (T is type) is alias of Result(T)
```

Type parameters use the same forms as generic functions: `T is type` for an unconstrained type and `T is type of Constraint` for a constrained type.

```first
AnimalList (T is type of Animal) is alias of T[]
```

Generic application uses parentheses, not angle brackets.

```first
users is Result(User)
```

Aliases are not functions. The header parameter list may contain only type parameters, such as `T is type` or `T is type of Constraint`. Runtime parameters belong in callable type expressions after `is alias of`.

## Callable Aliases

A callable type is written as `(parameters) is Result`.

```first
ReadInt is alias of () is int
Formatter is alias of (value is int) is string
```

Callable result types nest to the right.

```first
Factory is alias of () is () is int
```

Name callable types before forming arrays, nullable callables, or overload-like unions.

```first
readers is ReadInt[] = []
maybeReader is ReadInt or null = null
```

A union of callable aliases in callable position acts as a simple overload list. Calls resolve to the first callable member in alias order whose signature accepts the provided arguments.

```first
TextReader is alias of (value is int) is string
NumberReader is alias of (value is int) is int

Reader is alias of TextReader or NumberReader
readerFn is Reader

result = readerFn(1)
// Uses TextReader because it is first.
```

If an alias can be non-callable, call it only after narrowing to a callable type.

```first
MaybeReader is alias of TextReader or string
```

`MaybeReader` cannot be called until narrowed to `TextReader`.

Callable intersections are unsupported.

## Type Extraction

Type extraction carries type information from a statically known callable or class member into an alias.

```first
Result is alias of transformFn.return
Arguments is alias of transformFn.arguments
```

For callable values, `.return` extracts the callable result type and `.arguments` extracts the callable parameter signature.

```first
Transform is alias of (value is int) is string
transformFn is Transform = value => "converted"

TransformResult is alias of transformFn.return
TransformArguments is alias of transformFn.arguments
```

The same extraction works from a callable alias.

```first
AliasResult is alias of Transform.return
AliasArguments is alias of Transform.arguments
```

Extraction uses the finalized static signature. It does not inspect runtime closure captures.

For callable overload aliases, `.return` follows the same first-matching signature rule as invocation when enough argument context exists. Otherwise, narrow to one callable member before extraction. `.arguments` is available only when one callable member is known or every callable member has the same labels, types, optionality, and rest shape.

```first
TextReader is alias of (value is int) is string
NumberReader is alias of (value is int) is int
Reader is alias of TextReader or NumberReader

ReaderArguments is alias of Reader.arguments
// Valid: both callable members accept value is int.

ReaderResult is alias of Reader.return
// Needs call context or narrowing; do not infer a blind union.
```

If an alias is not always callable, `.return` and `.arguments` are unavailable until the type is narrowed to the callable part.

```first
MaybeReader is alias of TextReader or string
MaybeReaderResult is alias of MaybeReader.return
// Invalid until narrowed: MaybeReader is not always callable.
```

Default expressions are not part of callable type extraction. A defaulted implementation parameter appears as optional in its callable type, but executable defaults belong to the implementation.
