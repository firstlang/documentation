# Operator Overloading

Only scalar-backed primitive classes and unit declarations can overload operators.

Operator overloads are defined inside the primitive class or unit body. Extension classes cannot define operator overloads.

An operator overload starts with `operator`, followed by the operator token or fixed comparison pair, followed by the operand list.

```first
📏 cm is unit (
	operator + (this, that is cm) is cm (
		return cm(this + that)
	)
)
```

## Eligible Types

Operator overloads are allowed on unit declarations and on primitive classes whose root is a concrete primitive, a primitive group, or another scalar-backed primitive class.

Fresh aggregate primitive classes rooted directly in `primitive` cannot overload operators.

## Operands

Every operator overload declares exactly two operands. One operand is the bare token `this`. The other operand is `that is Type`.

The operand order is exact. The compiler does not assume that an operator is commutative.

```first
📏 cm is unit (
	operator - (this, that is i32) is cm (
		return cm(this - that)
	)
	
	operator - (that is i32, this) is cm (
		return cm(that - this)
	)
)
```

The first overload handles `cm - i32`. The second handles `i32 - cm`.

Operator overload result types follow the same inference rules as ordinary functions. A result annotation may be written, but it is not required.

## Overloadable Operators

The overloadable arithmetic operators are:

- `+`
- `-`
- `*`
- `/`
- `\`
- `%`
- `**`

The overloadable comparison forms are fixed pairs:

- `== !=`
- `< >=`
- `> <=`

The body defines the first operator in the pair. The second operator is the boolean negation of the body result.

```first
🪨 UserId is u64 (
	operator == != (this, that is UserId) (
		return this == that
	)
)
```

Standalone comparison overloads such as `operator ==`, `operator !=`, `operator <`, or `operator >=` are invalid. Reversed pair spellings such as `operator != ==` are invalid.

The compiler does not infer swapped operand order. `a > b` is not rewritten as `b < a`.

Strict equality (`===`), strict inequality (`!==`), unary operators, bitwise operators, logical operators, coalescing operators, ternaries, assignment, compound assignment, `key of`, and postfix `throw` are not overloadable.

Compound assignment uses the selected binary operator, then assigns the result back to the target. The result must be assignable to the target type.

Duplicate operator overloads with the same operator form and operand type order produce a compiler notice.

When no overload applies, ordinary unit adoption, same-unit defaults, numeric conversion, and built-in operator rules may still apply.
