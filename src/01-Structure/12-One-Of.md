# One Of

`one of` selects a named value, a literal value, or a value belonging to a listed type. Shared rules are in [[11-Selection-Types]].

## Named Entries

```first
Day is one of (
	monday
	tuesday
	wednesday
)
```

Bare runtime-style names declare entries, even when a local has the same name. With no explicit values, the values above are 1, 2, and 3.

Reserve explicit numeric values first, then number inferred entries above the highest explicit value in declaration order.

```first
Example is one of (
	a       // 3
	b       // 4
	c = 2
)
```

Explicit aliases within one body are allowed:

```first
HttpStatus is one of (
	ok = 200
	success = 200
	notFound = 404
)
```

ok and success compare equal but remain separate entries for iteration and display. Duplicate names receive notices. Collisions introduced through composition also receive notices; later entries win recovery.

## Unnamed Alternatives

```first
Width is one of (16, 32, 64)
Mixed is one of (1, "one", true)
Reply is one of ({ code = 1 }, { code = 2 })
Person is one of (User, Admin, null)
```

Literals constrain exact values. Types accept their instances. Type and literal alternatives may coexist; named entries cannot be mixed with unnamed alternatives. Spaces are not types. Arrays and objects must be exact, deeply immutable values.

Computation is excluded even if statically evaluable:

```first
Bad is one of (
	8 + 8 // Notice.
	32
)
```

## Qualified References

```first
Settings (
	small = 16
	large = 32

	Width is one of (
		this.small
		Settings.large
	)

	NamedWidth is one of (
		small = Settings.small
		large = Settings.large
	)
)
```

Width captures values without copying their names. NamedWidth explicitly creates names. Space-locals have qualified addresses; function-locals stay function-local. Qualification is independent of visibility. In space-owned code, this denotes the containing space; in instance code it denotes the instance. A selection entry list introduces no receiver. Outer spaces require named qualifiers; class instance members cannot be captured through class-name qualification.

## Composition

```first
Base is one of (
	a
	b
)

Extended is Base or one of (
	c = 10
	d
)
// Extended: a = 11, b = 12, c = 10, d = 13.

Combined is First or Second or Third
```

Explicit values stay fixed. Inferred entries are numbered across the resulting selection after all explicit values. Source declarations remain unchanged. Body spread is not supported.

## Access And Iteration

```first
Offset is one of (-1, 0, null)
Reply is one of ({ code = 1 })

run() (
	x = Offset.-1
	y = Offset.null
	z = Reply.{ code = 1 }
)
```

Literal member access is real syntax and must name an allowed value. Raw permitted literals are also accepted. Runtime values require proof of membership before use in a narrower selection.

Named selections yield name/value pairs. Unnamed selections yield their listed values or types, in order; types are not constructed during iteration. Author-defined functions and other member declarations are not allowed in the body.
