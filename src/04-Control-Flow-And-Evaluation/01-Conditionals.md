
# Conditionals

Conditionals work like TypeScript conditionals, except block bodies use `(` and `)` instead of `{` and `}`.

```first
if (condition1) (
	// Run code
)
else if (condition2) (
	// Run code
)
else (
	// Run code
)
```

The format is:

```first
if (<condition>) (
	<statements>
)
else if (<condition>) (
	<statements>
)
else (
	<statements>
)
```

`else if` and `else` branches are optional. An `else` branch, when present, must be the final branch.

Conditional bodies always require parenthesized blocks. The language does not support TypeScript-style braceless single-statement bodies.

```first
// ❌ Wrong
if (ready)
	start()
```

Use:

```first
if (ready) (
	start()
)
```

`if` is statement-only. It does not produce a value and cannot be used as an expression.

Use a ternary for two-arm expression selection. Use `matches` for multi-arm expression selection.

## Condition Types

An `if` condition must be either:

- a `boolean`
- a nullable value
- an expression that produces one of those types, such as a comparison or an `is` check

Unlike TypeScript and JavaScript, conditionals do not use general truthiness. Strings, numbers, arrays, objects, and other non-boolean values are not accepted as conditions merely because they could be interpreted as true or false.

```first
if (count) (
	// Invalid: numbers are not coerced to boolean
)

if (count !== 0) (
	// Valid
)
```

The editor may offer an expansion for numeric values in condition position. For example, placing a number in an `if` condition can be rewritten to an explicit comparison against zero.

```first
if (count) (
	// Editor can expand to:
)

if (count !== 0) (
	// Explicit numeric condition
)
```

The same rule applies to strings and collections. They require explicit checks.

```first
if (name !== "") (
	console.log(name)
)

if (items.length !== 0) (
	console.log(items)
)
```

## Nullable Conditions

Nullable values are allowed directly in condition position. The condition checks whether the condition expression itself is not `null`.

```first
if (user) (
	console.log(user.name)
)
else (
	console.log("No user")
)
```

Inside the truthy branch, the value is narrowed to its non-null type. Inside the `else` branch, the value is narrowed to `null` when the original type permits that conclusion.

```first
user is User or null

if (user) (
	// user is User here
	console.log(user.name)
)
else (
	// user is null here
)
```

A non-nullable value is not allowed directly in condition position just because it contains nullable fields.

```first
profile is Profile

if (profile) (
	// Invalid: profile itself is not nullable
)

if (profile.photo) (
	// Valid when profile.photo is nullable
)
```

Nullable values keep their type when assigned to a local. The local can then be checked directly.

```first
photo = profile.photo

if (photo) (
	// photo is the non-null photo type here
)
```

Nullable booleans follow the same rule: `false` is still a boolean value, not `null`.

```first
enabled is boolean or null

if (enabled) (
	// enabled is boolean here, and its value is true
)
else (
	// enabled is false or null here
)
```

Use an explicit comparison when the distinction matters.

```first
if (enabled !== null) (
	// enabled is boolean here
)
```

## Branch Evaluation

Branches are tested from top to bottom. The first branch whose condition passes is executed, and the remaining branches are skipped.

```first
if (status === "ready") (
	start()
)
else if (status === "waiting") (
	queue()
)
else (
	cancel()
)
```

This has the same control-flow behavior as a TypeScript `if` / `else if` / `else` chain.
