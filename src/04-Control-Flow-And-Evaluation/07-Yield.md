# Yield

`yield` emits one value without ending its enclosing loop or function. It has two destination kinds:

- A captured `each` or `loop`, where emitted values construct an immutable array.
- The current function, where emitted values make the function a generator. First calls these 🏓 ping-pong functions.

Standalone scopes, conditionals, `matches` expressions, and other non-iterative constructs are never yield destinations. Use `return` to provide the single result of a captured standalone scope or captured `matches` expression.

Every `yield` requires an explicit operand.

```first
yield // ❌ Wrong: a yielded value is required.
```

Typist may expand an incomplete `yield` by inserting `null`, but the resulting source is explicitly `yield null`.

# Loop Collection

The nearest captured loop is the default yield destination.

```first
positive = numbers each number (
	if (number > 0) (
		yield number
	)
)
```

A loop-targeting `yield`:

1. Evaluates its operand completely.
2. Appends the result to the loop's current immutable captured array.
3. Continues immediately after the `yield` without suspending execution.

The captured array's element type is the union of all values lexically yielded to that loop.

```first
values = items each item (
	if (item is TextItem) (
		yield item.text
	)
	
	if (item is NumberItem) (
		yield item.number
	)
)
```

Here, the captured element type is the union of the yielded text and number types. When lexical yield sites exist but none execute, the loop produces an empty array of that inferred element type.

## Current Capture

A named loop capture is visible inside its own body as the current accumulated array. It begins empty and reflects every value previously yielded to that loop.

```first
unique = items each item (
	if (!unique.includes(item)) (
		yield item
	)
)
```

The current capture is immutable. It cannot be reassigned or mutated from inside the loop. This allows the common build-then-freeze collection pattern without introducing a temporarily mutable source value.

When the current capture is emitted to the function, the consumer receives a stable immutable snapshot. Later loop yields create newer accumulated versions without changing an already exposed snapshot. The compiler may implement this through freezing, structural sharing, or copy-on-write storage without changing source semantics.

```first
snapshots(items is Item[]) (
	collected = items each item (
		yield item
		yield function collected
	)
)
```

## Breaking Collection

`break` never carries a value and never contributes to a captured loop.

To emit a final value and then terminate the loop, write both operations:

```first
values = items each item (
	yield transform(item)
	break
)
```

`return` also skips loops during destination lookup, including captured loops. A return inside a loop targets the nearest captured match, captured standalone scope, or current function. If it abandons the loop, ordinary iterator cleanup runs before the returned value reaches that destination.

# Yield Targets

Eligible yield destinations are:

- Captured enclosing loops.
- The current function as the terminal destination.

Uncaptured loops are not yield destinations. Conditional bodies, match arms, standalone scopes, calls, conditions, and ordinary grouping parentheses are not yield destinations.

An unqualified `yield` targets the nearest eligible destination. A targeted loop yield names a visible captured loop label before the operand. A function-targeting yield uses `yield function`:

- `yield value` targets the nearest eligible destination.
- `yield outer value` targets the visible captured loop labeled `outer`.
- `yield function value` targets the current function.

```first
produce() (
	outerValues = outerItems each outerItem as outer (
		innerValues = innerItems each innerItem as inner (
			yield innerItem
			yield outer transform(innerItem)
			yield function summarize(innerItem)
		)
	)
)
```

The three operations append to `innerValues`, append to `outerValues`, and emit from `produce`, respectively.

The function boundary is an eligible terminal destination rather than a boundary that invalidates lookup. A nested function, callback, or worker begins its own target sequence; `yield` never crosses into its enclosing function.

Typist manages each destination by identity and draws its visual connection.

An operation whose named loop target is not visible, not captured, or not a loop produces a prominent notice. The entire operation is cooked: its operand is not evaluated, it emits nothing, it does not suspend, and it does not make the function a generator. Execution continues with the following expression. The compiler never retargets an invalid operation to another destination.

# Ping-Pong Functions

A runnable `yield` lexically targeting the current function statically makes that function a generator, which First presents as a 🏓 ping-pong function. This does not depend on whether the yield executes at runtime. Loop-targeting and cooked yields do not affect the containing function's kind.

A function may also declare generator status in its signature without relying on an implementation body.

```first
values() is yield int
```

When a function has an implementation body, this explicit spelling is optional if a runnable function-targeting `yield` already infers the same generator kind. When the body is omitted, the `yield` annotation supplies the generator classification and element type.

Typist draws the 🏓 function treatment and a non-editable `yields` chip in its signature presentation. `returns` and `yields` are derived editor chips rather than serialized or editable keywords. The broader syntax for explicit and inferred function result types is defined with function signatures.

The underlying protocol uses the familiar `Generator(TYieldType)`, `Iterator`, and `Iterable` terminology. Ping-pong is the human-facing First name, as `one case of` is the human-facing name for an algebraic data type.

The `Yield` type is the union of the operands of every reachable function-targeting `yield`. Loop-targeting yields do not contribute to it. First generators have no separate consumer-input or final-return type parameters.

## Calling And Advancing

Calling a ping-pong function creates one suspended `Generator(TYieldType)` execution. The body does not begin until its first `next()` call.

Calling a bodyless ping-pong function creates an empty generator. Its first `next()` returns `done()` and emits no zero element.

The generator:

- Is both an `Iterator` and an `Iterable`.
- Returns itself from `iterator()`.
- Provides parameterless `next()` using the ordinary `value(...)` or `done()` iteration result.
- May be consumed directly with `each`.
- Represents one function invocation; another call creates an independent generator.
- Is stateful, sequential, and single-consumer.

Only one active owner may advance a generator. Gradual Ownership may transfer that ownership, but the generator cannot be duplicated or advanced concurrently. Re-entering `next()` while the same generator is executing is invalid and receives a prominent notice and safe recovery rather than overlapping execution.

A function-targeting `yield`:

1. Evaluates its operand completely.
2. Exposes the resulting value to the consumer.
3. Suspends immediately after the `yield`.

Resumption continues with the following expression. The `yield` operation itself does not receive or produce a resumption value and cannot be captured through assignment.

## Completion And Return

Reaching the end of a ping-pong function or executing a valueless function-targeting `return` completes the generator with plain `done()`. First generators do not have a separate final completion value.

When a function-targeting `return` carries an operand inside a ping-pong function, the operand is evaluated, Typist draws `🗑️`, the result is discarded, and the generator then completes. A possible low-priority notice for this code smell is reserved for future diagnostic policy.

To emit a final value and then complete, write both operations:

```first
yield value
return
```

`break` and `continue` retain their ordinary targets and never suspend the generator. Breaking a captured loop finalizes its current accumulated array and continues after that loop. Values already exposed to a generator consumer remain valid after any later control transfer.

A `return` captured by a nested match or standalone scope does not complete the generator. Return target rules are defined in [[06-Return]].

## ECMAScript Differences

First deliberately omits ECMAScript's bidirectional and final-value generator channels in its initial design:

- `next()` accepts no resumption value.
- `yield` does not evaluate to a consumer-supplied value.
- `return` does not produce a separately captured generator completion value.
- Generator cancellation uses the ordinary iterator cleanup `break()` rather than `return(value)` or `throw(value)` channels.

These omissions avoid coroutine-style runtime behavior and keep generators aligned with ordinary one-way iteration. They may be reconsidered if concrete demand appears, but they are not part of the initial Rust standard-library implementation.

## Suspension, Ownership, And Cleanup

Suspension preserves the generator's complete execution context, including locals, loop positions, captured-loop accumulators, active iterators, and held resources. Suspension itself performs no cleanup.

A yielded value must remain valid independently of the suspended generator. Gradual Ownership may move it, share immutable storage, clone it, or use reference counting. A yield must never expose a borrow that becomes invalid when the generator resumes or is destroyed.

Every generator provides the ordinary iterator cleanup `break()`. Calling it permanently abandons the suspended execution, cleans active inner iterators from innermost to outermost, releases retained state, and leaves the generator completed. Later `next()` calls produce `done()`. Early exit while consuming a generator through `each` invokes this cleanup automatically. Dropping the generator performs the same cleanup through Rust-backed destruction.

# Asynchronous Generators

The initial yield specification commits only synchronous `Generator(TYieldType)` behavior. A function combining `await` with function-targeting `yield` is expected eventually to produce an asynchronous generator consumed through `await each`, but its exact protocol and cancellation semantics are deferred until First's async, worker, and asynchronous-iteration model is finalized.
