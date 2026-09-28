# Return

`return` exits the nearest eligible single-result destination and supplies its value. A destination may be:

- A captured `matches` expression.
- A captured standalone scope expression.
- The current function.

`return` does not target loops. Captured loops construct arrays through `yield` rather than receiving one returned value.

```first
findItem(items is Item[]) is Item or null (
	items each item (
		if (isMatch(item)) (
			return item
		)
	)
	
	return null
)
```

Here, the loop is invisible to return-target lookup, so both operations target the function.

# Captured Destinations

A match or standalone scope becomes a return destination only when its resulting value is consumed by an enclosing expression.

```first
result = value matches (
	"ready" makeReadyResult()
	
	"pending"
		logPending()
		return makePendingResult()
)
```

The multi-expression arm uses `return` to supply `result`. The match arm itself is an incidental scope and does not intercept the operation.

An uncaptured match or standalone scope is invisible to return-target lookup.

```first
run(value is string) (
	value matches (
		"stop" return null
	)
	
	continueRunning()
)
```

Because the match is uncaptured, `return null` exits `run`.

Adding a capture intentionally makes the construct behave like an immediately evaluated single-result scope.

```first
result = value matches (
	"stop" return null
)
```

Here, the same unsuffixed spelling targets `result` instead.

Captured loops remain invisible even when they appear between a `return` and its destination.

```first
result = value matches (
	"search"
		collected = items each item (
			if (accept(item)) (
				return item
			)
			
			yield transform(item)
		)
		
		return fallback()
)
```

Both returns target the captured match. The first abandons the active loop, which performs its ordinary iterator cleanup. `collected` receives no value on that path because control leaves the match.

# Return Values

A valueless `return` supplies frozen `null` to an unannotated ordinary function, captured match, or captured standalone scope. A valueless `return` targeting an explicitly typed ordinary function supplies that function result type's zero value and produces a notice at the return site.

```first
result = (
	if (cancelled) (
		return
	)
	
	return makeResult()
)
```

The resulting type is the union of operands from all reachable returns targeting that capture. It includes `null` when a targeting return is valueless or when execution can reach the destination's end without supplying a value.

Captured matches additionally include reachable single-expression arm results. A multi-expression match arm that reaches its end without a targeting return supplies frozen `null`. See [[03-Matches]].

Function result inference includes only returns that target that function. Returns captured by nested matches or standalone scopes do not contribute to the function's result type.

When an explicitly typed ordinary function can reach its end without returning, fall-through supplies the declared result type's zero value silently. This also applies to omitted, empty, and anchor-only function bodies.

```first
label() is string (
	if condition (
		return "ready"
	)
)
// When condition is false, label returns "".
```

An explicit return operand that is incompatible with the destination type also produces a notice and recovers with that destination's zero value.

```first
label() is string (
	return 42 // Notice; returns "".
)
```

Fallback follows the actual return destination. A return captured by a nested match or standalone scope still uses that capture's inferred null-producing rules; a suffixed return that targets the function uses the function's declared zero fallback.

# Return Targets

`return` uses a zero-based outward target suffix:

- `return` is the canonical spelling of `return.0` and targets the nearest eligible destination.
- `return.1` targets the next eligible destination.
- `return.2` targets the destination outside that.

```first
resolve(value is string) is Result (
	local = value matches (
		"local" return makeLocalResult()
		"global" return.1 makeFunctionResult()
	)
	
	return normalize(local)
)
```

Inside the captured match, `return` supplies `local`, while `return.1` exits `resolve`.

Eligible captured matches and standalone scopes are counted from nearest to outermost. The current function is the terminal destination. Loops, uncaptured matches, uncaptured standalone scopes, conditional bodies, match arms, calls, conditions, and ordinary grouping parentheses do not count.

A nested function, callback, worker, or other function-like body starts its own target sequence. `return` never crosses that boundary into an enclosing function.

A suffix is a literal nonnegative decimal integer under the same grammar as `break`, `continue`, and `yield`. Typist manages the destination by identity, draws its visual connection, removes leading zeros, canonicalizes `.0` to the unsuffixed spelling, and rewrites the depth when connected code moves.

An operation whose requested depth has no destination produces a prominent notice. The entire return is cooked: its operand is not evaluated, no control transfer occurs, and execution continues with the following expression. The compiler never clamps an invalid depth to another destination.

# Iterator Cleanup

The operand of a valid `return` is evaluated completely while active iteration bindings and resources remain available. Cleanup begins only after operand evaluation finishes.

When a return abandons active `each` iterations before their iterators report `done`, the compiler invokes each iterator's qualifying cleanup `break()` exactly once. Cleanup runs from the innermost abandoned iterator to the outermost before the value reaches its destination. See [[04-Loops]].

# Ping-Pong Functions

A function-targeting `return` inside a 🏓 ping-pong function completes the generator with plain `done()`. First generators do not expose a separate final return value.

When such a return carries an operand, the operand is evaluated, Typist draws `🗑️`, the result is discarded, and the generator completes. A low-priority notice for this code smell is reserved for future diagnostic policy.

To emit one final value before completion, use both operations.

```first
yield value
return
```

A return captured by a nested match or standalone scope does not complete the generator. See [[07-Yield]].
