The `many of` feature is a selection type for choosing zero or more named identities as flags.

It is the First syntax for what other languages call a _flags enum_. The shared selection type rules are defined in [[Selection Types]].

Basic syntax:

```
Direction is many of (
	up
	down
	left
	right
)

foo(d is Direction) (
	console.log(d)
)

startup (
	foo(Direction.up)
)
```

Authors can choose to specify values, allow them to be inferred by the compiler, or mix explicit and inferred values. A named entry may also be assigned a composite of other entries from the same `many of` using `and`.

In the above case, the values are inferred by the compiler. Inferred values start at 1 and increment by powers of 2. Therefore, if the above values were to be explicit, they would be:

```
Direction is many of (
	up = 1
	down = 2
	left = 4
	right = 8
)
```

The recommended practice is to allow the compiler to infer the values unless:

- The author needs the `many of` values to start at a non-1 value
- The author needs to provide values that encode multiple values together (example below)

```
Direction is many of (
	up = 1
	down = 2
	vertical = 3
	left = 4
	right = 8
	horizontal = 12
)
```

Composite entries are written in terms of named entries rather than bitwise operations:

```
Interrupts is many of (
	keyboard
	mouse
	input = keyboard and mouse
	timer
	shutdown
)
```

The right-hand side of a composite assignment may contain only entries declared in the same `many of`, combined with `and`. It is evaluated as the bitwise union of those entries. A composite entry does not allocate a new bit; it receives the combined value of its components. Components may be ordinary flags or other composite entries, provided the references are acyclic. References to locals, parameters, other selection types, runtime expressions, and bitwise operators are not allowed.

Explicit values do not need to be powers of two, and they do not need to be clean composites of other declared flags.

When values are not ordinary powers of two or clean composites, the generated flag helpers may not behave the way the author expects. First still preserves the explicitly written values.

Reserve all explicit bits before inference within the current `many of` body, regardless of entry position. Inferred values begin at the power of two above the highest occupied explicit bit and advance by powers of two in declaration order. With no occupied explicit bits, begin at 1. This numbering is local to each declaration and is completed before the declaration is composed into another `many of`.

```
Direction is many of (
	up = 1
	down
	left = 8
	right
)
```

In this example, `down` infers to `16` and `right` infers to `32`.

# The Empty Flag State

Every `many of` has an empty flag state with the numeric value `0`. It is implicit and does not need to be declared. When no explicit zero-valued entry exists, the compiler exposes it as `Example.none`:

```
Direction.none
```

The empty state is also the zero value of the type. This means that an uninitialized `Direction` is valid and equal to `Direction.none`; it is not an invalid or exceptional value.

An author may give the empty state a domain-specific name by declaring exactly one entry with the explicit value `0`:

```
Interrupts is many of (
	noInterrupt = 0
	keyboard
	timer
	shutdown
)
```

In that form, `Interrupts.noInterrupt` is the zero value and the compiler does not also create `Interrupts.none`. An explicit zero-valued entry is therefore a name for the empty flag state, not an ordinary flag. More than one explicit zero-valued entry is invalid and produces a notice.

If no entry is assigned `0`, inferred flags begin at `1` and proceed through the powers of two. An explicit zero-valued entry consumes no inferred bit.

# Underlying Type of `many of` Definitions

A `many of` definition always fits into an unsigned integer, where that unsigned integer is sized to the smallest possible width to encapsulate the values, with a limit of `u128`.

All `many of` values must resolve to integer values. Other underlying value types are not allowed because the flag operations require bit math.

Raw integer values may contain bits that are not decomposable from the declared flags. The compiler may generate a notice for unknown bits, but that notice policy is not yet specified.

Explicit aliases and composite values within one original body remain allowed. When composing distinct flag sources, overlapping bits produce notices. Later conflicting entries win recovery; omit earlier conflicting entries in their entirety rather than stripping individual bits.

## Composition

```first
Permission is many of (
	read
	write
)

ExtendedPermission is Permission or many of (
	archive = 8
	share
)
// Permission: read = 1, write = 2.
// ExtendedPermission: read = 1, write = 2, archive = 8, share = 16.
```

Composition uses `or` in the header. Any number of flag sources may be combined. Body spreads are not allowed. Each source declaration resolves its explicit and inferred values independently before composition. Those values remain fixed in the composed selection. An inline body's inferred values are resolved locally using that body's explicit values and do not renumber entries imported from another source. Duplicate names and overlapping bits receive notices, with later entries winning recovery.

Author-defined functions are not permitted in a `many of` body. The operations below are built-in helpers, not author-defined members.

Each `many of` definition is packaged with a built-in API for performing the operations, which is more ergonomic than using standard bitwise operators `&` and `|` operations.

The list of "methods" available on each `many of` is as follows:

```first
// Returns true if this value contains the flag.
// Replaces: (target & flag) === flag
hasFlag(flag is TManyOf) is boolean

// Returns a new value with the flag added.
// Replaces: target | flag
addFlag(flag is TManyOf) is TManyOf

// Returns a new value with the flag removed.
// Replaces: target & ~flag
removeFlag(flag is TManyOf) is TManyOf

// Returns a new value with the flag's state inverted.
// Replaces: target ^ flag
toggleFlag(flag is TManyOf) is TManyOf

// Returns true if this value has no flags set (is 0).
hasNoFlags() is boolean

// Returns true if any of the provided flags are present in this value.
// Replaces: (target & (flagA | flagB)) !== 0
hasAnyFlag(...flags is TManyOf[]) is boolean

// Returns true only if every provided flag is present in this value.
hasAllFlag(...flags is TManyOf[]) is boolean

// Returns a list of numbers representing the individual flag values that make up this value.
// This handles overlapping values (like 'vertical = 3') by decomposing 
// based on the smallest power-of-two components. The empty flag state is not included in the result.
decomposeFlags() is number[]
```
