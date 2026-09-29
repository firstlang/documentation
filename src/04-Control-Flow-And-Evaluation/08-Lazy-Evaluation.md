# Lazy Evaluation

Lazy evaluation delays an initializer until the binding is first read. The result is then memoized and every later read returns the same value.

```first
value defer = getValue()
```

`defer` is written immediately before `=`. Without `defer`, the initializer is evaluated eagerly at the declaration point.

## Local Bindings

A local binding may be lazy:

```first
render() (
	value defer = getValue()

	if (shouldUseValue) (
		return value
	)

	return "fallback"
)
```

`getValue()` is not called when control reaches the declaration. It is called only if execution later reads `value`.

The initializer runs at most once for each execution of the scope that creates the binding. If the binding is never read, the initializer never runs.

## Fields

A stored field may also be lazy:

```first
Report (
	summary defer = buildSummary()

	print() (
		console.log(this.summary)
	)
)
```

Each instance memoizes its own field value. Reading `report.summary` for the first time evaluates `buildSummary()` for that instance. Later reads of the same field on the same instance return the memoized value.

Lazy field initializers follow ordinary field access rules. A protected lazy field remains protected; a public lazy field remains public.

## Result Type

A lazy binding has the same type it would have if the initializer were eager. If the declaration has an explicit type, the initializer must produce a value assignable to that type. If the declaration omits the type, the initializer determines it.

```first
count is int defer = expensiveCount()
name defer = loadName()
```

## Evaluation

The first read evaluates the initializer in the lexical context of the declaration. For a field, `this` is the instance whose field is being read.

Lazy initializers are synchronous. `await` is not valid inside a lazy initializer.

Lazy initializers follow the ordinary effect rules of their surrounding executable context. Prefer `defer` for values rather than hidden side-effect triggers.

If a lazy initializer bombs, the bomb is memoized as the binding's value. Later reads return the same bomb.

A lazy binding is initialized with once-only synchronization. If multiple threads read it before initialization completes, exactly one thread evaluates the initializer and the others wait. When evaluation completes, all waiting and future reads observe the same memoized value, including a bomb value.

## Use

Use `defer` when a value is expensive, may not be needed, or must be initialized later than its declaration position. Avoid it for literals and cheap expressions where eager evaluation is clearer.
