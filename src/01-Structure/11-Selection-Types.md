# Selection Types

First has three selection forms: `one of` (one named value, literal value, or value belonging to a listed type), `many of` (zero or more named flags), and `one case of` (one named case with its payload).

```first
Day is one of (
	monday
	tuesday
)

Width is one of (16, 32)
Person is one of (User, Admin, null)

Permission is many of (
	read
	write
)

Message is one case of (
	text(value is string)
	closed()
)
```

User and Admin denote class types. Person accepts their instances or null; declaring it does not construct instances. A space is not a type.

Selection declarations may be generic under [[02-Generics]].

## Entry Forms

A standalone runtime-style name declares a named entry. It never captures a same-named local. `name = value` supplies an explicit named value. A literal, qualified value reference, or type reference contributes an unnamed alternative. Named and unnamed entries cannot be mixed in a resulting selection; doing so produces a notice. Type and literal alternatives may be mixed.

```first
Settings (
	small = 16

	Named is one of (
		small
		large
	)

	Values is one of (
		Settings.small
		32
	)
)
```

Named.small is inferred as 1. Values contains 16 and 32 and does not acquire a member named small. Classification examines top-level entries only: a literal inside `name = { code = 1 }` does not make the entry unnamed.

## Composition

Compose declarations with `or` in the header. Any number of compatible sources may be listed, with an optional final inline selection body.

```first
Base is one of (
	a
	b
)

Extended is Base or one of (
	c
)

Combined is First or Second or Third

MorePermission is Permission or many of (
	archive
)

MoreMessage is Message or one case of (
	image(url is string)
)
```

Composition expands entries in source order, left to right. A referenced composition contributes its entries in that order. Preserve whether each value was explicit or inferred. Explicit values remain fixed; inferred values are assigned for the entire resulting selection and may differ from their source values. Source declarations remain unchanged. A numeric source entry does not silently convert to a destination's differently numbered entry.

Named selections compose with named selections, flags with flags, and cases with cases. Unnamed alternatives form ordinary unions of types and values. Incompatible selection forms produce a notice.

Selection bodies do not accept `...` spreads of selections, objects, or arrays. This applies to all three forms. Ordinary expression spread elsewhere is unaffected.

## Numbering

Reserve all explicit numeric values before assigning inferred entries, regardless of their positions. For `one of`, start above the highest explicit numeric value, then increment by one in expanded declaration order. Inferred values are integers; if explicit fractional values coexist with inferred entries, start at the smallest integer strictly above the highest explicit value. Non-numeric explicit values consume no numeric positions. With no explicit numeric values, begin at 1.

```first
Base is one of (
	a       // 3
	b       // 4
	c = 2
)

Extended is Base or one of (
	d = 10
	e       // 13
)
// Extended: a = 11, b = 12, c = 2, d = 10, e = 13.
// Base remains a = 3, b = 4, c = 2.
```

For `many of`, begin at the power of two above the highest bit occupied by explicit values, then double for each inferred entry. With no occupied explicit bits, begin at 1. The special none entry is zero and consumes no inferred position.

Numeric literals follow [[04-Provisional-Numbers]], not an unconditional platform-int default. Report overflow or unrepresentable values rather than wrapping.

## Collisions And Recovery

Conflicts produce notices, not normal overwrite semantics. Later conflicting entries win recovery in all selection forms; earlier conflicting entries are omitted from the recovered selection. Source declarations are not mutated. Reserve written explicit values before inference, including values involved in conflicts, so recovery does not create avoidable inferred collisions.

- Duplicate named entries conflict.
- Explicit aliases within a single named `one of` body may share values. Numeric collisions introduced by composing distinct source entries produce notices.
- Repeated unnamed values produce duplicate notices after normalization.
- Composed `many of` entries conflict on overlapping bits. Explicit composite flags within one original body remain supported; see [[15-Many-Of]].
- Duplicate `one case of` names conflict, even with identical payloads or shared origin. Diamond composition does not deduplicate cases. There are no implicit overloads or payload unions.

Ignore `nan`, `bomb`, and `unknown` alternatives with notices. A malformed declaration whose form cannot be established uses enum form for recovery. This does not resolve missing values or suppress their notices; unresolved values still follow [[01-Unknowable-Recovery]].

## Direct Values

Entries accept literals and direct references to immutable literal values. Arithmetic, calls, getters, and other computation cannot supply entries, even if compile-time evaluable. Moving computation into a binding does not bypass the rule. Direct-reference chains must terminate in permitted values and be acyclic. Forward references among direct space declarations are allowed; generated members are resolved after the relevant space seals.

Arrays and anonymous objects are exact deeply immutable values. Array order matters. Objects require exactly the same fields and recursively equal values, independent of field declaration order. Extra fields do not match. Mutable contents, cycles, instance captures, and functions are excluded. Listing a class type differs from capturing an instance as a constant.

Equality uses no string/number coercion or user-defined equality. Resolve provisional numeric representations losslessly before membership and duplicate checking. Positive and negative floating zero compare equal; NaN is excluded.

## Compatibility And Attestations

Selections are structurally typed. A selected value fits a target if all its possible values belong to the target. A larger set does not fit a smaller set without narrowing.

```first
Small is one of (16, 32)
Large is one of (16, 32, 64)

takeSmall(value is Small) ()
takeLarge(value is Large) ()

check(small is Small, large is Large) (
	takeLarge(small) // Accepted.
	takeSmall(large) // Notice: could be 64.
	if (large is Small) (
		takeSmall(large) // Accepted.
	)
)
```

Named declaration access surfaces also describe entry names and values. Extra members can satisfy a smaller member contract; that does not reverse selected-value compatibility. Equal value sets are compatible across named and unnamed forms. Case alternatives include names and compatible payload shapes.

`value is Selection` tests membership and narrows the successful branch, including exact aggregate membership and listed class types. See [[04-Attestations]].

## Access And Iteration

Named access uses `Day.monday`. Literal access includes `Width.32`, `Name."bob"`, `Offset.-1`, `Maybe.null`, `Reply.{ code = 1 }`, and `Sequence.[1, 2]`. The accessed value must belong to the selection. These are parsed source forms, not decorations.

Named selections iterate name/value pairs in declaration order, retaining aliases. Unnamed selections iterate their listed values or types in declaration order. Type alternatives yield types, not instances, under ordinary compiler-side rules. Ordering affects numbering and iteration, not set compatibility.

An empty `one of ()` is an empty set. It does not implicitly contain zero or null as a selection member. When code asks for a zero value of that invalid empty shape, First reports a notice and recovers with numeric zero.

For zero synthesis, a nonempty `one of` uses the first surviving expanded entry, `many of` uses the empty flag set, and `one case of` uses the first surviving case with zeroed payload fields. Ordinary unions choose `null` when it is admitted; otherwise they use the first written alternative's zero.

## Runtime Shape And Members

The declaration access surface need not exist at runtime. Selected values retain the scalar, aggregate, or sum representation they need; cases retain tags and payloads. Numeric values that must survive composition, serialization, or foreign interfaces must be explicit.

`one of` and `many of` cannot contain author-defined functions. No selection supports fields, properties, ordinary constructors, nested spaces, or ghosts. Case constructors are part of case syntax, not ordinary constructors. Built-in flag operations remain available.

The former `one value of` spelling is retired. Canonical source uses `one of` without legacy metadata. When updating old source, qualify captured space-local values rather than merely deleting the word value.
