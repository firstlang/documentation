# Operator Usage

Operators combine, compare, update, or choose between values. First keeps most familiar TypeScript operator behavior, but uses named operators where the symbolic form is too dense or conflicts with the language's readability goals.

For the complete operator list and precedence table, see [[06-Operators]] in the language reference.

## Arithmetic

Arithmetic operators work on numeric values and on other values that define the relevant operation.

```first
width = 12
height = 4

area = width * height
remaining = width - height
scaled = area / 2
```

First separates fractional division from integer division.

```first
items = 7
boxes = 2

average = items / boxes
// 3.5

fullBoxes = items \ boxes
// 3
```

Use `%` for remainders and `**` for exponentiation.

```first
index = 17 % 5
volume = side ** 3
```

First does not include `++` or `--`. Reassign explicitly when a mutable value needs to change by one.

```first
var count = 0

count += 1
```

## Assignment

`=` assigns a new value to a mutable binding, field, or indexed target.

```first
var retries = 0

retries = 1
```

Compound assignment combines an operation with assignment.

```first
var total = 10

total += 5
total *= 2
```

Compound assignment uses the same underlying operation as the non-assignment form. If a type customizes `+`, `+=` follows that behavior.

## Comparison

Comparison operators return `boolean`.

```first
age = 41

isAdult = age >= 18
isExact = age == 41
isDifferent = age != 0
```

Use comparisons explicitly. First does not treat numbers, strings, arrays, or objects as general truthy or falsy conditions.

```first
count = 3

hasItems = count != 0
```

## Logical Operators

Use `not` to negate a boolean.

```first
isReady = false

shouldWait = not isReady
```

Use `and` and `or` for boolean logic when both sides are boolean expressions.

```first
canSend = isReady and hasConnection
canRetry = hasRetries or isManualOverride
```

`and` and `or` short-circuit like JavaScript `&&` and `||`. The symbolic spellings still parse as legacy input, but tooling should rewrite them to `and` and `or`.

The same words are also used by coalescing forms in value-producing expressions. See [[02-Coalescing]] for fallback behavior such as `??`, `else`, and `catch`.

## Bitwise Operators

Bitwise operators operate on integer bits.

```first
read = 0b001
write = 0b010

readWrite = read | write
hasRead = (readWrite & read) != 0
flipped = readWrite ^ write
```

Selection types such as `many of` are usually clearer than raw bit masks. Use bitwise operators when the code is intentionally working with integer flags or binary data.

## Ternaries

The ternary operator chooses between two expressions from a boolean condition.

```first
label = count == 0 ? "Empty" : "Ready"
```

Use a ternary when both branches are ordinary values. Use coalescing when the choice is specifically about `false`, `null`, or bombs.

## Name Retrieval

`key of` returns the source name of a visible declaration or member as a string.

```first
Name = "Bob"

Animal (
	eat() (
		// ...
	)
)

animalName = key of Animal
methodName = key of Animal.eat
```

`key of` is closer to C# `nameof` than TypeScript `keyof`: it retrieves a name for runtime or tooling use, not a mapped-type key set.
