# Generics

Generics let one declaration work with several types while preserving static type information. First's generics are inspired by TypeScript, with a reduced feature set designed for intelligibility and straightforward lowering to statically compiled targets.

Classes, functions, selection types, and aliases may be generic.

## Generic Parameters

Generic parameters share the ordinary parameter list with runtime parameters. An uppercase-leading parameter is generic; a lowercase-leading parameter is a runtime parameter.

```first
identity(T, value is T) is T (
	return value
)
```

A bare generic parameter is unconstrained. This follows the ordinary rule that an omitted type annotation means `unknown`.

Annotate a generic parameter to constrain it. `TCol is Collection` is analogous to TypeScript's `TCol extends Collection`.

```first
first(T, TCol is Collection(T), values is TCol) is T (
	return values[0]
)
```

A type argument satisfies its constraint when it is assignable to the constraint under First's ordinary compatibility rules. Nominal classes use inheritance, aliases use their expanded structural types, selections use selected-value compatibility, and primitive groups use declared membership. A constraint may reference an earlier generic parameter.

Generic parameters must precede runtime parameters. A generic parameter may refer only to earlier generic parameters.

## Generic Functions

Function calls infer omitted generic arguments from runtime arguments first and the expected result type second.

```first
name = identity("Ada")
```

When inference cannot determine one unique satisfying type, supply generic arguments in the declared leading positions.

```first
empty = identity(string, "")
```

Generic arguments are compile-time arguments. They do not become runtime values.

A reference to a generic function preserves its generic signature. Assignment to a monomorphic callable type specializes it using the expected signature.

```first
IntIdentity is alias of (value is int) is int
intIdentity is IntIdentity = identity
```

Generic function expressions follow the same rules.

```first
identityFn = (T, value is T) is T => value
```

## Generic Classes

A class declares its generic parameters on its constructor. The parameters have declaration-wide type scope, including the class body, member signatures, member bodies, and inheritance clause.

```first
Box (
	constructor(T, value is T)
	value is T
)
```

A generic class must have an explicit constructor signature. Its body may be omitted under the ordinary constructor rules. A synthesized constructor may propagate an already-bound parent specialization, but it cannot introduce generic parameters.

In a type position, generic application specializes the class type.

```first
textBox is Box(string)
```

In an expression position, the same form constructs the specialized class. Construction may infer omitted type arguments from runtime arguments.

```first
explicitBox = Box(string, "hello")
inferredBox = Box("hello")
```

An inheritance clause may refer to generic parameters declared by the constructor. The compiler collects the constructor signature before resolving the class header.

```first
ChildBox is Box(T) (
	constructor(T, value is T)
)
```

A nested class does not implicitly receive an enclosing class's generic arguments. It declares and receives its own arguments.

## Generic Selection Types

Selection types declare generic parameters after the selection name and before `is`.

```first
Result(T, E) is one case of (
	ok(value is T)
	error(value is E)
)
```

Applications require exactly one explicit type argument per declared parameter. Missing, excess, and partial applications are invalid.

```first
result is Result(string, Error)
```

The same parameter rules apply to `one of`, `many of`, and `one case of` declarations.

## Generic Aliases

Aliases use the same generic parameter rules.

```first
Pair(T, U) is alias of {
	first is T
	second is U
}
```

Alias applications require every declared type argument and do not support partial application.

```first
pair is Pair(string, int)
```

See [[15-Aliases]] for alias type expressions and extraction.

## Compatibility And Editability

Immutable generic values are covariant across nominal subtype relationships.

```first
dogs is Dog[]
animals is Animal[] = dogs
```

`var` permits rebinding and does not change variance.

```first
animals is var Animal[] = dogs
animals = [Cat()]
```

An `editable` specialization is invariant wherever editing could replace a value involving the generic parameter.

```first
editableDogs is editable Dog[]
editableAnimals is editable Animal[] = editableDogs // Invalid.
```

An immutable or narrower specialization may widen into a new editable specialization through an independent copy or conversion.

```first
editableAnimals is editable Animal[] = editableDogs.copy()
```

Aliases remain transparent and use the compatibility of their expanded types. Selection specializations apply ordinary selection compatibility after substituting their generic arguments. Callable parameter and result compatibility follows the callable rules.

## Specialization

First monomorphizes each concrete generic application. Generic parameters and explicit generic arguments exist at compile time and are not passed or stored as runtime values. Each concrete nominal class specialization has distinct static type identity; aliases remain transparent.

```first
name = identity("Ada")
count = identity(1)
```

The compiler produces separate `string` and `int` specializations. Neither call passes a runtime type descriptor for `T`.

Generic type attestations such as `T is Animal` are not supported. Attest runtime values instead.

```first
inspect(T is Animal, value is T) (
	if (value is Dog) (
		useDog(value)
	)
)
```

## Type Extraction

A generic function must be specialized before its signature can be extracted. An expected callable alias can provide that specialization.

```first
StringIdentity is alias of (value is string) is string
stringIdentity is StringIdentity = identity
StringIdentityReturn is alias of stringIdentity.return
```

To keep the extracted alias generic, explicitly declare and forward its generic parameters through a generic callable alias.

```first
Identity(T) is alias of (value is T) is T
IdentityReturn(T) is alias of Identity(T).return
```

An extraction such as `identity.return` is invalid because `T` is unbound. A generic class may be specialized directly in an extraction target.

```first
BoxArguments is alias of Box(string).arguments
```

## Built-In Utilities

First provides compiler-implemented generic type transformations rather than exposing TypeScript-style mapped types.

```first
Partial(T)
Pick(T, K)
Omit(T, K)
Exclude(T, U)
Extract(T, U)
Capitalize(T)
Uncapitalize(T)
UpperCase(T)
LowerCase(T)
ToSnake(T)
FromSnake(T)
```

Generic application uses parentheses, never angle brackets. The utilities are built into the compiler and cannot be defined in ordinary First code.
