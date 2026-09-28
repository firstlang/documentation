# Numbers

First combines TypeScript-style numeric expressions with concrete Rust-style numeric representations. Most numeric literals begin provisionally and settle on a concrete primitive as their surrounding context becomes known. See [[04-Provisional-Numbers]].

`number` is an alias for `f64`. `numeric` is the annotation for a provisional number. They are not interchangeable.

## Integers

First provides five signed and five unsigned fixed-width integer types:

```first
i8
i16
i32
i64
i128

u8
u16
u32
u64
u128
```

Their widths, ranges, binary representations, and ordinary bitwise operations match the corresponding Rust primitives.

### Overflow

Fixed-width integer arithmetic uses two's-complement wrapping. Overflow never panics or terminates execution, and development and release builds produce the same value.

```first
value is u8 = 255
value += 1

console.log(value) // 0
```

Compiled Rust uses explicit wrapping operations where ordinary Rust operators could panic. It does not contain development-only overflow checks.

`declare noArithmeticOverflow` instructs the debugging interpreter to report arithmetic overflow within its scope. The operation still produces its normal wrapped result. This declaration affects interpreter diagnostics only and does not change compiled output.

### Bitwise Operations And Shifts

Bitwise operations use the concrete width of their operands. They do not coerce values to JavaScript's signed 32-bit representation.

Shifts lower to Rust's wrapping shift operations. A shift count at least as large as the operand width is reduced modulo that width. When `declare noArithmeticOverflow` is active, the debugging interpreter reports an oversized shift without changing its result.

## Fixed Decimals

A fixed decimal stores an integer together with a fixed number of decimal places. Its type name has the form `<backing-integer>.<scale>`.

For a type with scale `N`, the represented value is its backing integer divided by `10 ** N`.

```first
amount is i16.2 = 123.45
```

`amount` stores the backing integer `12345` in an `i16`.

The available signed fixed-decimal types are:

- `i8.1` through `i8.3`
- `i16.1` through `i16.5`
- `i32.1` through `i32.10`
- `i64.1` through `i64.19`
- `i128.1` through `i128.39`

The available unsigned fixed-decimal types are:

- `u8.1` through `u8.3`
- `u16.1` through `u16.5`
- `u32.1` through `u32.10`
- `u64.1` through `u64.19`
- `u128.1` through `u128.39`

Fixed decimals inherit the wrapping behavior of their backing integers.

```first
signed is i8.2 = 1.27
signed += 0.01
console.log(signed) // -1.28

unsigned is u8.2 = 2.55
unsigned += 0.01
console.log(unsigned) // 0.00
```

The common type for fixed-decimal arithmetic uses the widest operand backing type and the greatest operand scale.

```first
left is i32.4 = getLeft()
right is i64.2 = getRight()
result = left + right // i64.4
```

This rule does not widen the backing type merely to cover every value that could overflow while an operand is rescaled. Such overflow follows the ordinary wrapping rules.

When a fixed-decimal operation or conversion must discard fractional digits, it truncates toward zero. A potentially lossy implicit conversion receives a saw decoration. Rounding is requested explicitly; it is not controlled by a scoped declaration.

## Arbitrary-Precision Numbers

`big` is an arbitrary-precision signed integer. `big.1` through `big.64` are arbitrary-precision fixed decimals with the indicated scale.

- `big`
- `big.1` through `big.64`

These values allocate storage as needed. They do not overflow because of magnitude. First does not provide unsigned `big` variants.

## Floating-Point Numbers

First provides four IEEE 754 binary floating-point types:

```first
f16
f32
f64
f128
```

They respectively implement the IEEE 754 binary16, binary32, binary64, and binary128 formats, including signed zero, infinities, and `nan` values.

The special values are spelled in lowercase:

```first
nan
infinity
-infinity
```

A decimal-point literal is a provisional fixed decimal by default. A lowercase `f` suffix explicitly creates an `f32`; `ff` creates an `f64`.

```first
1.25	// Provisional fixed decimal
1.25f	// f32
1.25ff	// f64
```

## Division And Remainder

`/` is fractional division. Dividing integers with `/` does not silently perform Rust-style truncating integer division.

```first
5 / 2 // 2.5
```

`\` is integer division and truncates toward zero.

```first
5 \ 2 // 2
-5 \ 2 // -2
```

`%` computes the remainder. For signed operands, a nonzero result has the dividend's sign.

Division by zero never panics:

- Integer, fixed-decimal, and arbitrary-precision operations produce zero.
- Floating-point operations preserve IEEE 754 behavior and produce an infinity or `nan` as appropriate.
- Remainder by zero produces zero for non-floating numeric types.

`declare noDivisionByZero` instructs the debugging interpreter to report division or remainder by zero for every numeric family. It does not change the result, compiled output, or control flow.

## Platform Aliases

The following aliases are resolved at compile time from the compilation target's pointer width, matching Rust's `isize` and `usize` model:

- `int` is `i32` on 32-bit targets and `i64` on 64-bit targets.
- `uint` is `u32` on 32-bit targets and `u64` on 64-bit targets.
- `decimal` is `i32.4` on 32-bit targets and `i64.4` on 64-bit targets.
- `udecimal` is `u32.4` on 32-bit targets and `u64.4` on 64-bit targets.

Cross-compilation uses the destination target. The architecture of the machine running the compiler or the eventual host does not affect these aliases.

`number` is always an alias for `f64` and is not platform-dependent.

## Primitive Groups

Primitive groups describe families of built-in primitives. They are primarily used as primitive-class bases and extension targets.

- `AnyNumeric` contains every integer, fixed decimal, float, arbitrary-precision number, and `char`.
- `AnyInteger` contains every fixed-width integer, `big`, and `char`.
- `AnySignedInteger` contains the signed fixed-width integers and `big`.
- `AnyUnsignedInteger` contains the unsigned fixed-width integers.
- `AnyDecimal` contains every fixed decimal, float, and `big.N` type.
- `AnyFixedDecimal` contains every fixed-width fixed decimal.
- `AnySignedFixedDecimal` contains the signed fixed-width fixed decimals.
- `AnyUnsignedFixedDecimal` contains the unsigned fixed-width fixed decimals.
- `AnyFloat` contains `f16`, `f32`, `f64`, and `f128`.

Membership in a primitive group classifies a type but does not grant it additional operators. In particular, `char` belongs to `AnyNumeric` and `AnyInteger` for constraints and ranges, but characters do not support arithmetic without conversion to an integer.

## The Primitive Top Type

`primitive` is the top type of every built-in and user-defined primitive. A primitive class may inherit from a built-in primitive, a primitive group, another primitive class, or directly from `primitive` to define a fresh value representation. See [[07-Primitive-Classes]].

```first
writePrimitive(value is primitive) (
	console.log(value)
)

writePrimitive(12)
writePrimitive("hello")
```
