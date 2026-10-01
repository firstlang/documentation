Units are compile-time numeric tags with suffix syntax.

They are a language primitive for building unit libraries. A unit affects type checking, overload selection, completions, and lowering, but it does not allocate a wrapper or preserve a distinct runtime object. Lowered output stores and passes the underlying numeric representation required by context.

First does not include dimensional analysis, automatic conversion tables, unit cancellation, or compound unit generation as built-in language behavior. Libraries define those behaviors with functions and operator overloads.

## Declaration

A unit is declared with `is unit`.

```first
m is unit
cm is unit

distance = 10m
```

A unit does not declare a numeric base. The tagged expression keeps or resolves to the concrete numeric representation selected by ordinary numeric inference and surrounding context.

Units cannot inherit from other units, and other declarations cannot inherit from units.

```first
m is unit
surveyM is unit

// Invalid:
// surveyM is m
```

Unit declarations may use ordinary identifier casing, including conventional uppercase symbols.

```first
K is unit
Hz is unit
```

## Unit Bodies

A unit may have a body for functions and operator overloads. Fields, constructors, setters, and ghosts are not allowed. A unit without members may omit the body.

```first
m is unit

cm is unit (
	operator + (this, that is cm) is cm (
		return cm(this + that)
	)
)
```

Functions associated with a unit are resolved as compile-time member syntax on tagged values and lower to ordinary function calls with the unit value as the first argument.

```first
cm is unit (
	toM(this) is m (
		return m(this / 100)
	)
)

distance = 200cm
meters = distance.toM()
```

## Suffix Syntax

Unit suffix syntax applies only to numeric literals and has no space between the literal and the unit name.

```first
width = 10m
offset = -3m
temperature = 300K
```

Built-in numeric suffixes are recognized first. Any remaining suffix must resolve to an in-scope unit.

```first
precise = m(1.25ff)
```

Suffix syntax is not used for variables or expressions. Use unit application instead.

```first
size = getSize()
width = m(size)
```

## Unit Application

`Unit(expression)` applies a compile-time unit tag to a numeric expression. It is unit application, not an ordinary class constructor call.

```first
width = m(getWidth())
```

Unit application lowers to the expression's underlying numeric value after checks and overload selection. It does not run constructor code.

Validation, normalization, and conversion belong in ordinary functions or explicit operator overloads.

```first
requirePositiveMeters(value is m) is m (
	if (value < m(0)) (
		// library-defined notice or bomb behavior
	)
	return value
)
```

## Annotations

A unit may appear alone as a type annotation for a unit-tagged provisional numeric value.

```first
width is m = 10
```

This is equivalent for unit checking to:

```first
width is m = m(10)
```

API boundaries should pin the runtime numeric representation with `with`.

```first
measure(width is i64 with cm) (
)

value is i64 with cm = 10cm
provisional is cm = 10
```

`value is Unit` leaves the numeric representation provisional. `value is NumericType with Unit` fixes the runtime numeric representation and attaches the erased unit tag.

## Bare Number Adoption

When a bare number appears in a binary operation with a unit, the compiler adopts the bare number into the unit type.

```first
total = 10m + 2
```

This is equivalent for unit checking to:

```first
total = m(10) + m(2)
```

The same rule applies when the bare number is on the left.

```first
total = 2 + 10m
```

Bare number adoption applies to the overloadable binary arithmetic operators:

- `+`
- `-`
- `*`
- `/`
- `\`
- `%`
- `**`

Compound assignment operators use the same rule as their underlying binary operator.

```first
distance += 2
```

Equality operators do not use bare number adoption. Bitwise and logical operators are not overloadable.

## Unit Relationships

Distinct units are unrelated unless a function, conversion, or operator overload explicitly relates them.

```first
m is unit
surveyM is unit

distance = 10m + 5surveyM // compiler notice unless an overload defines this operation
```

Libraries may define conversions or cross-unit operations with unit functions and operator overloads.

```first
inch is unit (
	toCm(this) is cm (
		return cm(this * 2.54)
	)
)

cm is unit (
	operator + (this, that is inch) is cm (
		return cm(this + that.toCm())
	)
)
```

## Unit Algebra

First does not automatically create compound units.

```first
area = 10m * 10m
ratio = 10m / 2m
```

The compiler never synthesizes compound units, cancels units, or infers that `m / s` is `mps`.

Same-unit `+` and `-` are valid by default and return the same unit. Same-unit `*`, `/`, `\`, `%`, and `**` require explicit operator overloads unless the operation involves bare-number adoption from the other operand.

```first
sum = 10m + 5m
area = 10m * 5m // compiler notice unless an overload defines the result
```

If a library wants `m * m` to return an area unit, or `m / s` to return a speed unit, it defines those operator overloads.

```first
m2 is unit
s is unit
mps is unit

m is unit (
	operator * (this, that is m) is m2 (
		return m2(this * that)
	)

	operator / (this, that is s) is mps (
		return mps(this / that)
	)
)
```

## Equality

Loose equality between a unit value and a bare numeric value compares the underlying numeric values unless a unit overloads `== !=`.

```first
10m == 10 // true
```

Strict equality is checked statically before lowering and requires the same unit tag and compatible underlying numeric value.

```first
10m === 10 // false
10m === 10m // true
```
