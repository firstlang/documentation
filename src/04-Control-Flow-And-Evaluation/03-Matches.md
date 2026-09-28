# Matches

Case analysis uses the `matches` keyword. A match may run as a standalone statement or produce a captured value.

```first
name matches (
	"alice" callAlice()
	"bob" callBob()
	else callNobody()
)
```

```first
result = name matches (
	"alice" makeAliceResult()
	"bob" makeBobResult()
	else makeNobodyResult()
)
```

The matched expression is evaluated exactly once before any patterns are tested. Arms are tested from top to bottom, and only the first matching arm runs.

# Patterns

The initial pattern language deliberately permits only statically analyzable forms:

- Literal values.
- Statically resolvable compile-time constants, including selection entries.
- `one case of` case patterns.
- Numeric and character range patterns with statically resolvable bounds and steps.
- `else`.

Arbitrary locals, calls, arithmetic, and other runtime expressions are not patterns.

```first
Day is one of (
	monday
	tuesday
)

day matches (
	Day.monday handleMonday()
	Day.tuesday handleTuesday()
)
```

```first
limit = calculateLimit()

value matches (
	limit handleLimit() // ❌ Wrong: runtime locals are not patterns
)
```

Literal and constant patterns use strict value equality equivalent to `===`. Pattern matching performs no runtime coercion and does not invoke user-defined equality overloads.

Primitive values, selection values, `one case of` values, and values accepted by range patterns may be matched. Array and anonymous-object literals are also valid exact-value patterns. They use the same deeply immutable structural equality as selection membership: arrays compare in order, objects require exactly the same fields and values. These patterns do not introduce bindings or perform partial destructuring.

```first
Reply is one of ({ code = 1 }, { code = 2 })

show(reply is Reply) (
	reply always matches (
		{ code = 1 } showOne()
		{ code = 2 } showTwo()
	)
)
```

The match above is exhaustive. An object with extra fields does not match either literal.

A pattern incompatible with the matched expression's type receives a prominent notice and is cooked.

# Arm Selection

Patterns may overlap. The first matching arm wins, so authors may intentionally place a specific pattern before a broader one.

```first
value matches (
	1 to 10 handleSmall()
	1 to 100 handleLarger()
)
```

A pattern wholly covered by earlier patterns receives a prominent unreachable-pattern notice and is cooked. Identical patterns after the first receive the same treatment. A partially overlapping pattern receives no notice because its ordering may be intentional.

When no arm matches, a standalone match continues with the expression after the block. A captured match produces frozen `null`.

An empty match is valid. It evaluates the matched expression once and then follows the ordinary unmatched behavior. Typist gives it a low-priority notice because it has no arms.

## `else`

`else` matches every value not selected by an earlier arm and must be the final reachable arm.

```first
value matches (
	1 handleOne()
	else handleOther()
)
```

An arm after `else` receives a prominent unreachable-arm notice and is cooked.

# Arm Bodies

A match arm may contain one expression or multiple expressions.

An arm containing exactly one top-level body expression supplies that expression as its implicit result. In a standalone match, the value is discarded under the ordinary discarded-value rules.

```first
result = value matches (
	"a" makeA()
	"b" makeB()
)
```

The expression may be written beside the pattern or indented beneath it. Expression count, not placement, determines whether the result is implicit.

```first
result = value matches (
	"a"
		makeA()
)
```

An arm containing multiple top-level body expressions is a statement body. It supplies a value only through a `return` targeting the captured match. Reaching the end of the body without such a return supplies frozen `null`, regardless of the final operation's own type. In a standalone match, the values of its ordinary expressions are discarded.

```first
result = value matches (
	"a" makeA()
	
	"b"
		doFirst()
		return makeB()
)
```

Typist draws a non-editable implicit-`return` marker beside the sole expression in a single-expression arm. A multi-expression statement body has no such marker and must contain an explicit serialized `return` to supply a result. Changing the number of expressions changes the arm form and receives immediate visual feedback.

Match-arm bodies are incidental scopes. They are neither independent `break` targets nor independent `return` targets. A captured match itself is eligible for both operations.

```first
result = value matches (
	"a"
		if (cancelled) (
			break
		)
		
		return makeA()
)
```

Here, `break` exits the captured match with frozen `null`, while `return` supplies the ordinary captured result. An explicit standalone parenthesized scope inside the arm remains an independent `break` target and becomes an independent `return` target when captured.

In an uncaptured match, `return` does not target the match. It continues outward to the nearest captured single-result construct or the current function. See [[06-Return]].

# Multiple Patterns Sharing One Body

Same-indentation patterns without bodies join the next pattern that has a body.

```first
value matches (
	"a"
	"b"
	"c" handleABC()
	"d" handleD()
)
```

This creates one logical arm for `"a"`, `"b"`, and `"c"`. Selecting any of those patterns runs `handleABC()` exactly once. It is not sequential fall-through execution.

The shared body may instead be indented beneath the final pattern.

```first
result = value matches (
	"a"
	"b"
		doFirst()
		return makeAB()
)
```

A bare pattern at the end of the block has no following body. It remains matchable, receives a prominent notice, and follows the resultless-arm behavior: no action in a standalone match and frozen `null` in a captured match.

Commas and newlines are interchangeable separators between match entries.

```first
result = value matches (
	"a", "b", "c" makeABC()
	"d" makeD()
)
```

Comments and blank lines are trivia and do not alter pattern grouping. Once an arm expression begins, newlines inside its syntactic delimiters follow the ordinary parser rules and do not create match entries. Formatting must preserve the semantic connection between each pattern group and its body.

The restricted pattern grammar gives the parser an unambiguous boundary between a pattern and a same-line body expression. No serialized arrow or separator is required.

# Captures And Result Types

A match is captured when its resulting value is consumed by an enclosing expression, including an initializer, argument, operator, or outer `return`. A match used as a standalone statement is uncaptured.

The resulting type of a captured match is the union of all reachable arm-result types.

The union includes:

- The expression type of every reachable single-expression arm.
- Operand types from reachable `return` operations targeting the match.
- Bomb types returned to the match.
- Frozen `null` when the match may remain unmatched, a multi-expression arm may reach its end, or a `break` may target the match.

A `return` targeting an outer capture or function contributes its operand type to that destination rather than to the match.

`null` is omitted when control-flow analysis proves that every completing path supplies a non-null result. See [[05-Break-and-Continue]] and [[06-Return]].

# Exhaustiveness Checking

`always matches` opts into exhaustiveness checking.

```first
letter always matches (
	"a" handleA()
	"b" handleB()
)
```

The compiler must prove that the preceding patterns cover every possible value of the matched expression. A coverage hole produces a prominent notice. Typist displays an injected `else null`, and the compiler emits that recovery so execution remains safe.

For a captured match, an injected fallback adds `null` to the resulting type. For a standalone match, its value is discarded and execution continues after the block.

Exhaustiveness is optional for both captured and standalone matches. Ordinary `matches` never requires complete coverage.

The compiler may prove coverage through:

- An `else` arm.
- Both boolean values.
- Every distinct value of a finite selection type, including exact aggregate literals. Named aliases with equal values require only one covered value.
- Every constructor of a `one case of` type using payload-wide patterns.
- Complete coverage of a literal union.
- Literal and range coverage of every value of any fixed integer or `char` type.

Floating-point domains do not receive range-based exhaustiveness proofs. Open domains such as unrestricted strings require `else`.

# One Case Of Patterns

When the matched expression has a `one case of` type, patterns may name its case constructors.

```first
message matches (
	default() showDefault()
	text(🥚 value) showText(value)
	image(🥚 url) showImage(url)
	reaction(":)") showSmile()
)
```

A supplied payload position may contain:

- A literal.
- A statically resolvable compile-time constant.
- A nested case pattern.
- A new binding rendered by Typist with the `🥚` decoration.

An ordinary identifier does not create a binding. Under the static pattern rules, it must resolve to a compile-time constant.

```first
message matches (
	text(value) showText(value) // ❌ Wrong: `value` is not a constant or new binding
)
```

Case patterns may omit any number of trailing payload positions. Omitted payloads are ignored.

```first
result matches (
	failed(500, "timeout") handleTimeout()
	failed(500) handleAny500Failure()
	failed() handleAnyFailure()
)
```

This omission is pattern-only behavior. Construction still requires every mandatory constructor argument.

Every supplied payload must correspond positionally to a declared payload. A pattern with too many payloads receives a prominent notice and is cooked. Each new binding must be unique within its pattern and is scoped to the shared arm body.

When multiple patterns share one body, they must introduce the same binding names with compatible types. Otherwise, the incompatible patterns receive prominent notices and are cooked. First does not initially provide a wildcard or discard pattern; an unused new binding is removed from generated code.

# Range Patterns

Integer and character ranges may be used as patterns.

```first
amountText = amount matches (
	1 til 30 "low"
	30 til 70 "med"
	70 to 100 "high"
)
```

`to` remains inclusive and `til` remains exclusive. A range pattern respects its step and matches only when `range.contains(subject, true)` succeeds.

```first
value matches (
	0 to 10 step 2 handleEven()
	else handleOther()
)
```

Bounds and steps must be statically resolvable. The subject and range types must be strictly compatible after provisional-number pinning; matching performs no runtime coercion. Floating-point range patterns are not supported.

Overlapping ranges follow the ordinary first-pattern-wins rule. Empty or fully shadowed range patterns receive prominent notices and are cooked. Exhaustiveness analysis accounts for inclusivity, boundaries, direction, and step alignment.

Typist presents match patterns and bodies in aligned columns. It draws `⏵` after patterns connected to a body and `⏷` after preceding patterns that share the following body. These are editor presentations rather than serialized source tokens.
