# Identifier Constraints

First identifiers follow JavaScript identifier rules. A valid identifier uses the same start and continuation rules as JavaScript, and it must not be a reserved word in the position where it appears.

The current parser may still use a narrower early implementation. The language rule is the JavaScript rule.

Identifier casing is part of First's syntax. Uppercase-leading identifiers belong to the structural family. Lowercase-leading identifiers belong to the runtime family. The parser and editor use this distinction to classify source forms, not merely to style-check them afterward.

## Lowercase Names

Lowercase-leading names are used for runtime values, members, and labels.

This includes:

- functions and class-contained functions
- fields and properties
- locals and loop bindings
- function and constructor arguments
- anonymous object keys
- labels used by `break`, `continue`, and loop-targeting `yield`
- entries inside selection types
- built-in primitive type names

```first
displayName(user is User) is string (
	label = user.name
	return label
)

User (
	name is string

	get displayName() is string (
		return name
	)
)

point = { x is int = 10, y is int = 20 }
```

Primitive type names such as `string`, `int`, and `boolean` are lowercase because that spelling is built into the language.

## Uppercase Names

Uppercase-leading names are used for structural declarations and structural bindings.

This includes:

- spaces
- classes
- extension classes
- primitive classes
- one of, one case of, and many of declarations
- aliases
- generic type parameters
- import names and import aliases

```first
App (
	User (
		name is string
	)

	Result(T, E) is one case of (
		ok(value is T)
		error(value is E)
	)

	UserId is alias of string
)

import Postgres as Database
```

## Selection Entries

The selection declaration name is structural and therefore uppercase. Entries inside the selection are runtime names and therefore lowercase.

```first
Direction is many of (
	up
	down
	left
	right
)

Message is one case of (
	text(value is string)
	closed()
)
```

Literal alternatives and type alternatives are not identifier declarations.

```first
Width is one of (16, 32, 64)
Person is one of (User, Admin, null)
```

## Units

Unit definitions may use either casing. This allows conventional unit symbols without creating a separate naming family.

```first
m is unit
cm is unit
K is unit
Hz is unit
```

## Qualified Names

Qualified names are made from separate identifiers joined by dots. The dot is not part of the identifier.

```first
App.Backend.User
user.displayName()
```

Named declarations cannot use dotted names in their declaration header. Write the nesting explicitly instead.

```first
App (
	Backend (
		User (
		)
	)
)
```

## Classification

Casing affects how a form is read. An uppercase call-shaped declaration is structural. A lowercase call-shaped declaration is a function.

```first
DisplayName (
)

displayName() (
)
```

The first form is read as a structural declaration. The second form is read as a function declaration
