# Loops

First has two loop forms:

- `each` iterates a value that provides the compiler-recognized iteration protocol.
- `loop` repeats indefinitely without an iterable.

First does not provide separate `for`, `while`, or `do` statements.

# `each`

`each` is a contextual suffix applied to an iterable expression.

```first
<iterable-expression> each <bindings> (
	<body>
)
```

```first
getItems() each item index source (
	console.log(item)
)
```

The iterable expression is evaluated exactly once. The compiler obtains one iteration context from that value and advances it until the context reports that iteration is done.

## Iteration Bindings

Every iterable defines an ordered iteration-binding contract. Each identifier following `each` binds one position from that contract.

Bindings are separated by whitespace. Commas are not used.

```first
items each item index source (
)
```

Binding names do not determine their meaning. Position determines meaning. The names declared by an iteration result are documentation and editor suggestions only.

Loop headers do not destructure the values occupying those positions. When an array item is itself a tuple, the first binding receives the whole tuple.

```first
pairs each pair index source (
	left right = pair
)
```

Changing the binding names does not turn the header into destructuring:

```first
pairs each left right (
	// left is the whole pair; right is the array index.
)
```

Destructuring remains a separate operation inside the loop body and follows the one-level destructuring rules defined in [[05-Destructuring]].

### Omitting Bindings

Bindings form a positional prefix. Any number of trailing bindings may be omitted, including all of them.

```first
items each item (
	console.log(item)
)

items each (
	performOneStep()
)
```

First has no discard placeholder. When an author needs a later position but not an earlier one, the earlier position is still bound and may remain unused.

```first
items each unusedItem index (
	console.log(index)
)
```

Unused bindings are removed when they are not needed by generated code.

### Too Many Bindings

Bindings beyond the iterable's maximum supported position are ignored and receive compiler notices. Typist renders these tokens with strike-out styling.

```first
items each item index source extra another (
	console.log(item)
)
```

The supported prefix binds normally and the loop still runs. Ignored tokens do not declare frozen-null locals. A later reference to one follows the ordinary unresolved-identifier recovery rules.

## Built-In Binding Contracts

Built-in iterables use two broad conventions:

- Sequence-like iterables begin with the current value and then provide positional or source context.
- Key-value iterables begin with the key and value and may then provide the source.

These are standard-library conventions, not restrictions on custom iteration contracts. A custom result may expose any positional payload shape. Binding names remain non-semantic.

### Arrays

Array iteration mirrors the callback arguments of JavaScript's `Array.prototype.forEach()`.

```first
items each item index array (
)
```

- `item` is the current element.
- `index` is its zero-based array index and has type `uint`.
- `array` is the array being traversed.

For an array produced by a transformation, the index and source belong to the transformed array rather than describing provenance in an earlier source.

### Ranges

Ranges mirror the array binding shape.

```first
1 to 10 step 2 each value index range (
)
```

- `value` is the current range value.
- `index` is its zero-based iteration index and has type `uint`.
- `range` is the range being traversed.

Range direction comes entirely from the range's `step`. An omitted step is positive `1`; direction is not inferred from the relative boundary values. Descending iteration therefore requires a negative step.

```first
10 to 0 step -1 each value (
	console.log(value)
)
```

The complete range rules are defined in [[08-Ranges]].

### Maps

Map iteration uses key-value order and exposes the source Map last.

```first
map each key value map (
)
```

This keeps the conventional key-value order used when destructuring JavaScript Map entries while also mirroring the availability of the source Map in `Map.prototype.forEach()`.

### Sets

Sets expose the current value and source Set. They do not synthesize an index.

```first
set each value set (
)
```

A Set has no intrinsic positional identity. In particular, an ordinal would be misleading when insertion-order preservation is disabled.

### Strings

Ordinary strings and byte strings define their binding contracts in [[06-Strings]]. Their Unicode and byte iteration rules are not generalized by this document.

## Iteration Order

`each` preserves the order produced by the iterator. The compiler does not independently reorder or parallelize loop iterations.

- Arrays iterate in ascending index order.
- Ranges iterate in step order.
- Maps and Sets iterate in insertion order by default.
- Map and Set construction may opt out of insertion-order preservation for improved performance. Their order is unspecified when that option is used.
- Custom iterators define their order through `next()`.

Unspecified order is still valid iteration. It does not mean undefined behavior.

## Immutable Collection Iteration

A collection cannot be mutated while it is being traversed by `each`. A mutable collection may be traversed read-only; mutation during that traversal is not supported.

When mutation is required, authors iterate an independent range and perform indexed operations explicitly.

```first
0 til items.length each index (
	// Mutate items through explicit indexed operations.
)
```

Ownership of the iterable and produced values is inferred from surrounding use under the ordinary Gradual Ownership rules. `each` does not introduce source-level borrowing annotations or a separate ownership policy. See [[17-Ownership]].

Later use can inform that inference:

```first
items = getItems()

items each item (
	console.log(item)
)

console.log(items.length)
```

A temporary iterable may instead be consumed when surrounding use permits it:

```first
getItems() each item (
	store(item)
)
```

## Binding Scope

Iteration bindings are immutable locals whose scope is the loop body.

```first
items each item index (
	console.log(item)
	console.log(index)
)
```

They may shadow outer immutable locals under the ordinary shadowing rules.

```first
item = fallback

items each item (
	console.log(item)
)

console.log(item)
```

They cannot shadow mutable locals and cannot be reassigned.

```first
🐝 item is var = fallback

items each item ( // ❌ Wrong: mutable locals cannot be shadowed
)
```

```first
items each item (
	item = replacement // ❌ Wrong
)
```

# The Iteration Protocol

Iteration uses compiler-recognized shapes rather than declared interface conformance. This is similar to the way a qualifying `toString()` is discovered through ordinary member lookup.

The concrete result types normally live in the standard library or beside a custom iterable. By convention, the standard library uses concrete specialized result types named `MapIterationResult` or `PairIterationResult` rather than one universal result type. This is not a language invariant, and result-type names have no compiler significance.

## Iteration Results

A qualifying iteration-result type is a `one case of` with exactly two compiler-significant cases:

```first
ItemIterationResult is one case of (
	value(item is Item, index is uint, source is Items)
	done()
)
```

- `value` may contain zero or more payload fields.
- Its payload fields define the positional iteration-binding contract.
- `done` has no payload.
- The result type and payload fields may have any names.
- Additional, missing, renamed, or malformed cases do not qualify.

The integer discriminant or other representation used to lower these cases is an implementation detail of `one case of` values.

## Iterable And Iterator Members

An iterable provides `iterator()`. The value returned from `iterator()` provides `next()`.

Conceptually:

```first
Items (
	constructor
	
	iterator() is ItemIterator (
		// Return a new iteration context.
	)
)

ItemIterator (
	constructor
	
	next() is ItemIterationResult (
		// Return value(...) or done().
	)
)
```

A qualifying `iterator()` must:

- Be accessible at the loop site.
- Be callable without arguments and therefore have no required parameters.
- Resolve without an overload ambiguity.
- Return a value with a qualifying `next()`.

A qualifying `next()` follows the same lookup and callability rules and must return a qualifying iteration-result type. Optional parameters do not prevent either member from qualifying because both remain callable without arguments.

The compiler performs these steps:

1. Evaluate the iterable expression once.
2. Call `iterator()` once to obtain the iteration context.
3. Call `next()` once for each attempted iteration.
4. Run the body for `value(...)` and bind its payload positions.
5. Stop at `done()` without running the body and without calling `next()` again.

### Omitted Payload Work

Omitting trailing loop bindings does not change the source-visible iterator protocol. A custom iterator still returns its complete declared `value(...)` payload.

Built-in iterables may use compiler-internal implementations specialized by the requested binding count. These implementations are not overloads visible to First code. They allow generated code to avoid calculating or packaging unused trailing values.

A production build may make similar eliminations for custom iterators when whole-program analysis proves them safe. This is a permitted optimization, not a language guarantee.

### Iterators Are Not Automatically Iterable

A value with only `next()` is an iterator, but it cannot be used directly with `each`.

```first
iterator each value ( // ❌ Wrong: iterator() is missing
)
```

Requiring `iterator()` establishes an explicit iteration boundary and prevents `each` from silently continuing an existing stateful context.

An ordinary iterable produces an independent iteration context for each `iterator()` call.

An iterable iterator may deliberately return itself from `iterator()`:

```first
ItemIterator (
	constructor
	
	iterator() is ItemIterator (
		return this
	)
	
	next() is ItemIterationResult (
		// Advance this iteration context.
	)
)
```

This stateful behavior must be explicitly provided; it is never inferred merely from the presence of `next()`. Reusing such an iterable iterator continues from its current position.

### Generators

A function containing a function-targeting `yield` produces a compiler-provided `Generator(TYieldType)`. A generator is a stateful iterable iterator: `iterator()` returns the same generator, and parameterless `next()` resumes it until the next function-targeting `yield` produces `value(yielded)` or the function completes with `done()`.

Generators are sequential and single-consumer. They have no consumer-input or final-return channel. Their complete suspension, ownership, targeting, and cleanup rules are defined in [[07-Yield]].

### Early Cleanup

An iterator may provide a compiler-recognized `break()` function for early cleanup.

```first
ItemIterator (
	constructor
	
	break() (
		releaseResources()
	)
)
```

The compiler calls iterator `break()` exactly once when control leaves that iteration before `done()`, including a targeted `break`, a `return` to any destination outside the iteration, or another control transfer that crosses the iteration. When one transfer abandons multiple active iterators, their cleanup runs from innermost to outermost. A local `continue` does not call cleanup because its iteration remains active.

The cleanup function follows the same accessibility, callability, and ambiguity rules as the other protocol members. Its result is ignored. Compiled execution silently discards that result. The debugging interpreter may eventually provide a low-priority notice when cleanup produces a meaningful discarded value, but that diagnostic is not guaranteed.

## Invalid Iterables

Iteration follows an "iterator or it did not happen" rule.

If the source type does not provide a qualifying iteration protocol, the entire loop expression is cooked:

- The iterable expression is not evaluated.
- No protocol member is called.
- The loop body does not execute.
- Typist marks the complete loop scope as unrunnable.
- A captured cooked loop produces frozen `null`.
- The compiler produces a notice at the loop boundary.

This recovery keeps a loop safe when, for example, a called function is refactored to return a non-iterable value.

## Union-Typed Iterables

Every member of an iterable source union must independently provide a qualifying iteration protocol. A nullable iterable therefore does not qualify until it is narrowed.

At runtime, the source's active union member selects the corresponding `iterator()` implementation. That implementation is still selected and called only once for the loop.

```first
items is Item[] or null

items each item ( // ❌ Wrong
)
```

```first
if (items) (
	items each item (
	)
)
```

When union members provide iteration results with different payload lengths, First creates the longest positional envelope.

Given three result types whose `value` cases are:

```first
value(first is A, second is B, third is C)
value(first is D, second is E)
value(first is F)
```

the combined loop bindings have these types:

```first
first is A or D or F
second is B or E or null
third is C or null
```

At each position, the compiler unions every type provided there and adds `null` when any union member omits that position. At runtime, an absent position receives frozen `null`. A binding after the longest payload is an ignored extra binding and receives the ordinary notice.

# Loop Expressions And Captures

Loops are expressions. They may appear uncaptured or in any expression position that captures their result, including a local initializer or function argument.

```first
captured = getItems() each item (
	yield item
)
```

```first
consume(
	getItems() each item (
		yield item
	)
)
```

`yield` sends values to the targeted loop capture and constructs an array. The captured array's element type is the union of every value lexically yielded to that loop. If those yield sites exist but none execute, the result is an empty array of that inferred element type.

The named capture is visible inside its own loop body as the current immutable accumulated array. It begins empty, reflects prior yields to that loop, and cannot be reassigned or mutated. Exposing the current capture outside the loop produces a stable immutable snapshot.

An uncaptured loop does not construct a result array and is skipped during yield target lookup. A yield inside it targets the nearest captured outer loop or, when none exists, the current function.

All loops are skipped during return-target lookup, including captured loops. `return` supplies one value to the nearest captured match, captured standalone scope, or current function; it never replaces or appends to a loop's captured array.

A captured loop containing only ordinary body expressions has no captured value. Its capture receives frozen `null` and a notice because the capture is pointless.

```first
captured = getItems() each item (
	console.log(item)
)
```

The complete targeting and control-transfer rules are defined in [[07-Yield]], [[05-Break-and-Continue]], and [[06-Return]].

# `loop`

`loop` creates indefinite repetition without constructing a range or iterator.

```first
loop (
	performWork()
)
```

It may expose one counter binding:

```first
loop index (
	console.log(index)
)
```

The counter:

- Has type `uint`.
- Begins at `0`.
- Increments by `1` before the next iteration begins.
- Uses the ordinary wrapping semantics of fixed-width integers.

On a 32-bit target, it wraps after `u32.max`; on a 64-bit target, it wraps after `u64.max`. Authors needing a different counter representation maintain it explicitly.

`loop` supports no other bindings. Extra bindings are ignored and receive notices without cooking the loop.

The counter follows the same body scope and immutability rules as an `each` binding. Extra bindings receive the same Typist strike-out treatment as extras after `each`.

`loop` is compiler-provided control flow. It does not call `iterator()` or `next()`.

# Asynchronous Iteration

The asynchronous spelling is `await each` and retains First's expression-first order.

```first
stream await each value (
	consume(value)
)
```

Each iteration waits for the next asynchronous result before running the body. Iterations remain sequential and ordered; `await each` does not imply concurrent fetching or concurrent body execution.

This document commits the `await each` syntax and its sequential behavior. Asynchronous generators are expected eventually to use `await each`. Their exact protocol and interaction with workers and cancellation are deferred until First's asynchronous and threading model is finalized.
