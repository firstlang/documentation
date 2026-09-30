# Break And Continue

`break` exits an enclosing breakable construct. `continue` advances an enclosing loop to its next iteration. Neither operation carries a value, and neither targets beyond the current function-like boundary.

Outward control transfer uses labels introduced with `as` on loops and other eligible targets. Typist may render those connections visually, but the source form names the target directly.

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

An unqualified `break` targets the nearest breakable construct.

An outward `break` names a visible breakable target:

```first
outerItems each outerItem as outer (
	innerItems each innerItem as inner (
		if (shouldStopEverything(innerItem)) (
			break outer
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

A target name must resolve to a visible breakable construct. Incidental conditional bodies, match arms, uncaptured matches, and other non-breakable scopes are not valid targets.

Typist draws a visual connection from each targeted `break` to its destination.

An operation whose named target is not visible or is not breakable produces a prominent compiler notice. The operation is cooked, no control transfer occurs, and execution proceeds with the following expression. The compiler never retargets an invalid operation to another destination.

Target lookup stops at the current function-like boundary.

## Iterator Cleanup

When `break` abandons active `each` iterations before their iterators report `done`, the compiler invokes each iterator's qualifying cleanup `break()` exactly once. Cleanup runs from the innermost abandoned iterator to the outermost. The targeted iterator is included when the destination is an `each` loop.

```first
outerItems each outerItem as outer (
	middleItems each middleItem as middle (
		innerItems each innerItem as inner (
			if (shouldStop(innerItem)) (
				break outer
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

An unqualified `continue` targets the current loop. An outward `continue` names a visible loop target:

```first
rows each row as outer (
	row each cell as inner (
		if (skipRow(cell)) (
			continue outer
		)
	)
)
```

Intermediate conditional, match, and standalone scope bodies are not valid `continue` targets. Typist manages and draws the destination loop using the same visual connection behavior as `break`.

A valid `continue` performs the targeted loop's ordinary advancement exactly once. An `each` loop requests its next iterator result. A `loop` counter increments once before its next iteration begins.

`continue` does not invoke cleanup for its targeted iterator because that iteration context remains active. When an outward `continue` abandons inner active iterators, their cleanup runs exactly once each from innermost to outermost before the targeted loop advances.

Target lookup stops at the current function-like boundary. Invalid-target recovery is the same as for `break`.
