# Functions And Calls

## Composing Small Functions

This example shows declarations, nested calls, and intermediate values.

```first
square(value is int) is int (
	return value * value
)

distanceSquared(x is int, y is int) is int (
	xSquared = square(x)
	ySquared = square(y)
	return xSquared + ySquared
)
```

## Optional And Default Arguments

This function accepts both a default value and an optional fallback.

```first
clamp(value is int, minimum is int = 0, maximum is int = 100) is int (
	if (value < minimum) (
		return minimum
	)
	
	if (value > maximum) (
		return maximum
	)
	
	return value
)
```

## Rest Arguments

This example collects an arbitrary number of integer arguments.

```first
sum(...values is int[]) is int (
	total is var int = 0
	
	values each value (
		total += value
	)
	
	return total
)

sampleTotal() is int (
	return sum(3, 5, 8, 13)
)
```

## Layering Function Calls

This example keeps a nested calculation readable by naming its intermediate results.

```first
clamp(value is int, minimum is int = 0, maximum is int = 100) is int (
	if (value < minimum) (
		return minimum
	)
	
	if (value > maximum) (
		return maximum
	)
	
	return value
)

percentage(part is int, whole is int) is int (
	return part * 100 / whole
)

completionLabel(completed is int, total is int) is string (
	value = percentage(completed, total)
	bounded = clamp(value)
	return `{bounded}% complete`
)
```
