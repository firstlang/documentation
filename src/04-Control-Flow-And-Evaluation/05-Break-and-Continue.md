# Break And Continue

`break` exits an enclosing breakable construct. `continue` advances an enclosing loop to its next iteration. Neither operation carries a value, and neither targets beyond the current function-like boundary.

Numeric target suffixes are First's source representation of labeled control transfer. Typist manages the underlying target connection by identity and rewrites the suffix when code moves.

# Break

`break` may target:

- An `each` or `loop` loop.
- An explicit standalone parenthesized scope, whether captured or uncaptured.
- A captured `matches` expression.

The following constructs are not independently breakable:

- `if`, `else if`, and `else` bodies.
- Individual `matches` arms.
- Uncaptured `matches` blocks.
- Function, method, callback, worker, class, and type bodies.
- Parentheses used for calls, conditions, or ordinary expression grouping.

An explicit standalone scope inside one of these constructs remains independently breakable. A function-like body instead establishes a target-lookup boundary.

```first
items each item (
	if (finished(item)) (
		break
	)
)
```

`break` never accepts an operand.

```first
break value // ❌ Wrong: break does not carry a value
```

When an operand is attached through manually edited or generated source, the compiler emits a prominent notice and cooks the operand without evaluating it. The valid `break` still performs its control transfer.

## Effects On Captures

Breaking a captured `matches` expression or captured standalone scope causes that construct to produce frozen `null`.

```first
result = (
	if (cancelled) (
		break
	)
	
	return makeResult()
)
```

Breaking a captured loop terminates iteration and finalizes the immutable array accumulated by earlier `yield` operations. `break` never adds an element to that array.

```first
values = items each item (
	if (finished(item)) (
		break
	)
	
	yield transform(item)
)
```

To add a final element and then terminate a captured loop, use both operations.

```first
yield finalValue
break
```

Single-result values are supplied with `return`, while captured-loop elements and generator emissions are supplied with `yield`. See [[06-Return]] and [[07-Yield]].

## Break Targets

`break` uses a zero-based outward target suffix:

- `break` is the canonical spelling of `break.0` and targets the nearest breakable construct.
- `break.1` targets the next enclosing breakable construct.
- `break.2` targets the breakable construct outside that.

Suffixes count only valid `break` targets. Incidental conditional bodies, match arms, uncaptured matches, and other non-breakable scopes do not affect the depth.

```first
outerItems each outerItem (
	innerItems each innerItem (
		if (shouldStopEverything(innerItem)) (
			break.1
		)
	)
)
```

The unqualified operation inside an incidental conditional still targets its nearest enclosing breakable construct.

```first
items each item (
	if (shouldStop(item)) (
		break
	)
)
```

A suffix is a literal nonnegative decimal integer. Signs, fractions, identifiers, and computed expressions are not suffixes. Typist removes leading zeros and canonicalizes `.0` to the unsuffixed spelling. There is no language-level maximum target depth.

Typist draws a visual connection from each `break` to its destination. The numeric suffix is the serialized depth of that editor-managed connection, not a target authors ordinarily maintain by hand. When connected code moves, Typist preserves the target by identity and rewrites the suffix.

An operation whose requested depth has no target produces a prominent compiler notice. The operation is cooked, no control transfer occurs, and execution proceeds with the following expression. The compiler never clamps an invalid depth to another target.

Target lookup stops at the current function-like boundary.

## Iterator Cleanup

When `break` abandons active `each` iterations before their iterators report `done`, the compiler invokes each iterator's qualifying cleanup `break()` exactly once. Cleanup runs from the innermost abandoned iterator to the outermost. The targeted iterator is included when the destination is an `each` loop.

```first
outerItems each outerItem (
	middleItems each middleItem (
		innerItems each innerItem (
			if (shouldStop(innerItem)) (
				break.2
			)
		)
	)
)
```

Cleanup runs for the inner, middle, and outer iterators in that order. See [[04-Loops]].

# Continue

`continue` skips the rest of the current iteration and advances the targeted loop exactly as if its body had reached its end normally.

```first
items each item (
	if (shouldSkip(item)) (
		continue
	)
	
	consume(item)
)
```

`continue` uses the same zero-based suffix spelling, but its targets are loops only:

- `continue` is the canonical spelling of `continue.0` and targets the current loop.
- `continue.1` targets the next outer loop.
- `continue.2` targets the loop outside that.

```first
rows each row (
	row each cell (
		if (skipRow(cell)) (
			continue.1
		)
	)
)
```

Intermediate conditional, match, and standalone scope bodies do not affect `continue` suffix counting. Typist manages and draws the destination loop using the same identity-based behavior as `break`.

A valid `continue` performs the targeted loop's ordinary advancement exactly once. An `each` loop requests its next iterator result. A `loop` counter increments once before its next iteration begins.

`continue` does not invoke cleanup for its targeted iterator because that iteration context remains active. When an outward `continue` abandons inner active iterators, their cleanup runs exactly once each from innermost to outermost before the targeted loop advances.

Target lookup stops at the current function-like boundary. The suffix grammar and invalid-target recovery are the same as for `break`.
