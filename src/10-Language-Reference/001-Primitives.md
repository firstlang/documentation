# Primitives

First's built-in primitive types are immutable value types. This page is the complete inventory of the concrete primitives defined by the language.

## Integer Types

### Signed Integers

```first
i8
i16
i32
i64
i128
```

### Unsigned Integers

```first
u8
u16
u32
u64
u128
```

## Fixed Decimals

A fixed decimal uses an integer type as its backing representation. Its name has the form `<integer-type>.<decimal-places>`.

### Signed Fixed Decimals

- `i8.1` through `i8.3`
- `i16.1` through `i16.5`
- `i32.1` through `i32.10`
- `i64.1` through `i64.19`
- `i128.1` through `i128.39`

### Unsigned Fixed Decimals

- `u8.1` through `u8.3`
- `u16.1` through `u16.5`
- `u32.1` through `u32.10`
- `u64.1` through `u64.19`
- `u128.1` through `u128.39`

The maximum scale is chosen so that the scaled value remains representable by the backing integer. Fixed decimals use the wrapping behavior of that integer.

## Floating-Point Types

```first
f16
f32
f64
f128
```

These are IEEE 754 binary floating-point types.

## Variable-Width Numbers

`big` is an arbitrary-precision signed integer. `big.1` through `big.64` are arbitrary-precision fixed decimals.

First does not provide unsigned `big` types.

## Other Primitive Types

```first
boolean
string
byte string
char
null
```

`string` contains Unicode text. `byte string` contains bytes. `char` contains one Unicode scalar value. `boolean` contains `true` or `false`, and `null` is the null value.

## Platform Aliases

These names are aliases, not additional primitive types:

- `int` is `i32` on 32-bit targets and `i64` on 64-bit targets.
- `uint` is `u32` on 32-bit targets and `u64` on 64-bit targets.
- `decimal` is `i32.4` on 32-bit targets and `i64.4` on 64-bit targets.
- `udecimal` is `u32.4` on 32-bit targets and `u64.4` on 64-bit targets.
- `number` is always an alias for `f64`.

