# One Of

`one of` describes either a named finite selection or a finite set of literal values. Shared rules are in [[11-Selection-Types]]. A declaration uses one form for its entire body; named entries and literal entries cannot be mixed.

## Named Entries

```first
Day is one of (
	monday
	tuesday
	wednesday
)
```

Bare runtime-style names declare entries, even when a local has the same name. With no explicit values, the values above are 1, 2, and 3. A named body may contain bare names, named assignments, or both.

Reserve explicit numeric values first, then number inferred entries above the highest explicit value in declaration order.

`none` is a special optional entry. When it appears in a named `one of`, it is always the zero value and is semantically moved to the front of the selection. Writing it anywhere other than first produces a notice, but the declaration remains usable and the recovered selection places `none` first. `none` does not accept an assigned value; `none = 0` and `none = anyOtherValue` both produce notices, because `none` is intrinsically zero.

When `none` is absent, the first surviving entry is the zero value. This allows selections to choose whether their default should be an explicit empty state or an ordinary domain value.

```first
Day is one of (
	none
	monday
	tuesday
)
// Day.none is the zero value.

Status is one of (
	ready
	failed
)
// Status.ready is the zero value.

LateNone is one of (
	ready
	none
)
// Notice: none should be first. Recovery moves none to the front.
```

```first
Example is one of (
	// > 3
	a
	// > 4
	b
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

## Literal Values

```first
Width is one of (16, 32, 64)
Mixed is one of (1, "one", true)
Reply is one of ({ code = 1 }, { code = 2 })
Maybe is one of (null, "value")
```

Literal entries constrain exact values. A literal may be a string, number, boolean, `null`, unit literal, regular-expression literal, array literal, or object literal. Array and object literals must contain only literals recursively and must be deeply immutable. 

Named and literal entries cannot be mixed:

```first
Invalid is one of (
	ready
	"ready"
)
```

Computation is excluded even if statically evaluable:

```first
Bad is one of (
	// > Notice.
	8 + 8
	32
)
```

## Named Constant Assignments

```first
Settings (
	small = 16
	large = 32

	NamedWidth is one of (
		small = Settings.small
		large = Settings.large
	)
)
```

The right-hand side of a named assignment may refer directly to a compiler-known constant through an explicit qualified name. The assignment creates the selection entry name; it does not create an unnamed value entry.

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
