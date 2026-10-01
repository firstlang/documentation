Units are numeric-backed primitive classes with suffix construction syntax.

They are a language primitive for building unit libraries. First does not include dimensional analysis, automatic conversion tables, unit cancellation, or compound unit generation as built-in language behavior. Those features can be defined by libraries using primitive classes and operator overloads.

## Declaration

A unit is a primitive class marked with `declare unit`.

```first
m is AnyNumeric (
	declare unit
)

distance = 10m
```

Units must be based on a concrete numeric primitive, a numeric primitive group, or another numeric-backed primitive class. Units cannot be based directly on `primitive`, because aggregate primitive classes do not have a single numeric value to pass as the implicit construction value.

`declare unit` is not inherited automatically. A derived primitive class is usable as a unit suffix only if it also declares `unit`.

Unit classes are not sealed by default.

## Suffix Syntax

Unit suffix syntax has no space between the numeric literal and the unit name.

```first
width = 10m
offset = -3m
```

Suffix syntax is constructor sugar for numeric literals only.

```first
width = 10m
width = m(10)
```

These two forms are equivalent.

Suffixes use in-scope unqualified unit names. They follow the same identifier rules as type names and may be uppercase.

Suffix syntax is not used for variables or expressions. Use normal construction instead.

```first
size = getSize()
width = m(size)
```

## Constructors

A unit class uses the same implicit first construction parameter as other primitive classes based on concrete primitives or primitive groups.

The unit suffix form can only target unit classes whose construction is satisfied by that implicit value. Unit constructors may still validate or normalize the value.

```first
positiveM is AnyNumeric (
	declare unit
	
	constructor() (
		if (this < 0) (
			// compiler notice or library-defined validation behavior
		)
	)
)
```

## Bare Number Adoption

When a bare number appears in a binary operation with a unit, the compiler adopts the bare number into the unit type.

```first
total = 10m + 2
```

This is equivalent to:

```first
total = m(10) + m(2)
```

The same rule applies when the bare number is on the left.

```first
total = 2 + 10m
```

This is equivalent to:

```first
total = m(2) + m(10)
```

Bare number adoption applies to the overloadable binary operators:

- `+`
- `-`
- `*`
- `/`
- `%`
- `**`

Compound assignment operators use the same rule as their underlying binary operator.

```first
distance += 2
```

This is treated through the same unit adoption behavior as `distance + 2`.

Equality operators do not use bare number adoption. Bitwise and logical operators are not overloadable.

## Assignment

When a bare number is assigned to a location with a known unit type, the compiler adopts the number into that unit type.

```first
width is m = 10
```

This is equivalent to:

```first
width is m = m(10)
```

The annotation supplies the unit target.

## Related Units

When a binary operation combines units in the same primitive-class inheritance chain, both operands are adopted into the most-derived unit type involved in the operation.

```first
m is AnyNumeric (
	declare unit
)

msv is m (
	declare unit
)

distance = 10m + 5msv
```

`msv` represents `SurveyMeter`, a domain-specific meter unit used for survey-grade measurements.

The operation is equivalent to:

```first
distance = msv(10) + msv(5)
```

The result type is `msv`.

A value of a more-derived unit may still be assigned to an ancestor unit type through the normal primitive-class ancestor relationship.

```first
distance is m = 10m + 5msv
```

## Unrelated Units

Binary operations do not work across unrelated unit types unless an operator overload defines the operation.

```first
m is AnyNumeric (
	declare unit
)

kg is AnyNumeric (
	declare unit
)

value = 10m + 5kg // ❌ compiler notice
```

Libraries may define conversions or cross-unit operations with normal primitive-class operator overloads.

```first
inch is AnyNumeric (
	declare unit
)

cm is AnyNumeric (
	declare unit
	
	operator + (this, that is inch) (
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

By default, these operations follow the unit's primitive-class numeric behavior. If a library wants `m * m` to return an area unit, or `m / m` to return a bare number, it defines those operator overloads.

```first
m2 is AnyNumeric (
	declare unit
)

m is AnyNumeric (
	declare unit
	
	operator * (this, that is m) (
		return m2(this * that)
	)
)
```

## Equality

Loose equality compares the underlying data unless a unit type overloads `== !=`.

```first
10m == 10 // true
```

Strict equality requires the exact unit type and underlying value to match.

```first
10m === 10 // false
```
