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

Authors can choose to specify values, allow them to be inferred by the compiler, or mix explicit and inferred values.

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

Explicit values do not need to be powers of two, and they do not need to be clean composites of other declared flags.

When values are not ordinary powers of two or clean composites, the generated flag helpers may not behave the way the author expects. First still preserves the explicitly written values.

Reserve all explicit bits before inference, regardless of entry position. Inferred values begin at the power of two above the highest occupied explicit bit and advance by powers of two in declaration order. With no occupied explicit bits, begin at 1.

```
Direction is many of (
	up = 1
	down
	left = 8
	right
)
```

In this example, `down` infers to `16` and `right` infers to `32`.

# `none` Values

The reason inferred values start at 1 and not 0 is because 0 represents the empty flag state. If the author wishes to start at 0, they need to specify their own values.

Explicit 0 values other than `none` are allowed. This can be useful for backend, hardware, or foreign API interop where 0 has a domain-specific meaning. It is not recommended for ordinary flags because it can make `hasNoFlags` confusing.

`many of` definitions don't allow a `none` value by default. They need to be provided explicitly. The following are equivalent:

```
Direction is many of (
	none
	up
	down
	left
	right
)
```

```
Direction is many of (
	none
	up = 1
	down = 2
	left = 4
	right = 8
)
```

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
// ExtendedPermission: read = 16, write = 32, archive = 8, share = 64.
```

Composition uses `or` in the header. Any number of flag sources may be combined. Body spreads are not allowed. Explicit values remain fixed, while inferred values are numbered across the whole resulting selection after all explicit bits. Source declarations remain unchanged. Duplicate names and overlapping bits receive notices, with later entries winning recovery.

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
// Returns true even when no named none entry exists.
hasNoFlags() is boolean

// Returns true if any of the provided flags are present in this value.
// Replaces: (target & (flagA | flagB)) !== 0
hasAnyFlag(...flags is TManyOf[]) is boolean

// Returns true only if every provided flag is present in this value.
hasAllFlag(...flags is TManyOf[]) is boolean

// Returns a list of numbers representing the individual flag values that make up this value.
// This handles overlapping values (like 'vertical = 3') by decomposing 
// based on the smallest power-of-two components. If the many of allows a
// none value, this isn't included in the set
decomposeFlags() is number[]
```
