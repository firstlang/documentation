# Scope Expressions

An executable parenthesized scope may appear in an expression position and have its result captured.

```first
result = (
	return "complete"
)
```

Scope expressions do not implicitly return their final expression. A value reaches the capture only through a `return` targeting that scope.

```first
result = (
	"discarded"
	return "captured"
)
```

Here, `"discarded"` has no effect on the capture and `result` receives `"captured"`.

A captured scope is both a `return` target and a `break` target. `return` exits it with a supplied value. `break` exits it with frozen `null`.

```first
result = (
	if (cancelled) (
		break
	)
	
	return makeResult()
)
```

The resulting type includes the operands of all reachable returns targeting the scope. It also includes `null` when a targeting return is valueless, a targeting break is reachable, or execution can reach the end normally.

An uncaptured standalone scope remains a `break` target but is invisible to `return` target lookup. A return inside it continues outward to the nearest captured single-result construct or the current function.

```first
run() (
	(
		return "function result"
	)
)
```

Explicit standalone scopes are distinct from incidental bodies belonging to conditionals and matches. Incidental bodies do not independently intercept `break` or `return`.

```first
result = (
	if (condition) (
		return "early"
	)
	
	return "normal"
)
```

Both operations target the captured outer scope. Parentheses used for function calls, conditions, or ordinary expression grouping are not executable scope targets.

The complete target and suffix rules are defined in [[05-Break-and-Continue]] and [[06-Return]].
