# Spread

Spread expands one value into another expression context.

```first
values = [1, 2, 3]
moreValues = [0, ...values, 4]
```

For ordinary value expressions, spread follows the JavaScript and TypeScript value-side model. It can be used where the surrounding expression accepts expanded array, object, or argument entries.

## Arrays

Array spread copies the source array's entries into the new array at that position.

```first
prefix = [1, 2]
suffix = [4, 5]
values = [...prefix, 3, ...suffix]
```

Spreading an array does not make the source array editable, and it does not preserve identity with the source array.

## Objects

Object spread copies fields into a new object literal.

```first
base = { name = "Ada", active = true }
user = { ...base, active = false }
```

Later fields with the same name replace earlier fields in the resulting object.

## Calls

Call spread expands an array into positional arguments.

```first
values = [1, 2, 3]
total = sum(...values)
```

The expanded values are checked against the callee's ordinary parameter list.

## Rest Parameters

In a parameter list, `...name is T[]` declares a rest parameter. A rest parameter collects remaining positional arguments into an array.

```first
sum(...values is int[]) is int (
	total is var int = 0
	
	values each value (
		total = total + value
	)
	
	return total
)
```

A function can have at most one rest parameter, and it must be the final parameter.
