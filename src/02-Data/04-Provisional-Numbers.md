# Provisional Numbers

`numeric` is First's provisional numeric annotation. It allows the compiler to select and revise a concrete numeric primitive as information becomes available from literals, operations, assignments, arguments, and return values.

`numeric` is not a runtime numeric type. Every provisional number has a compiler-known concrete representation before it is lowered.

`number` has a different meaning: it is an alias for the concrete `f64` type.

```first
provisional is numeric = 1.25
floatingPoint is number = 1.25
```

The first value begins as a fixed decimal. The second value is an `f64`.

## Literal Inference

An unsuffixed whole-number literal begins with the narrowest integer type that can represent its value. Positive values prefer unsigned integers; negative values require signed integers.

```first
count = 0
console.log(getPrimitiveTypeName(count)) // u8
```

An unsuffixed literal with a decimal point begins as the narrowest fixed decimal that can represent the written value.

```first
amount = 1.25
console.log(getPrimitiveTypeName(amount)) // u8.2
```

A lowercase `f` suffix pins a literal to `f32`. A lowercase `ff` suffix pins it to `f64`.

```first
single = 1.25f
double = 1.25ff
```

## Progressive Resolution

The compiler revises a provisional number when later operations require another concrete representation.

```first
x is var numeric = 0
x += getInt16()

console.log(getPrimitiveTypeName(x)) // i16
```

```first
x is var numeric = 0
x += getInt16()
x += getFloat32()

console.log(getPrimitiveTypeName(x)) // f32
```

This is a compile-time process. The runtime value does not carry a tag saying that it is provisional, and its primitive type does not change while the program executes.

When control-flow paths require different representations, the compiler first selects a common type that accepts every path without loss. If no such concrete type exists, the joined value uses a sum type rather than silently discarding information.

## Pinning

Context can pin a provisional number to a concrete type.

```first
small is u8 = 10
ratio is f32 = 1 / 3
price is decimal = 19.99
```

Stored fields, serialized layouts, and foreign interfaces must have a concrete representation by the time they are lowered. A `numeric` parameter or return is specialized to the concrete representation required by its call site.

If the surrounding program does not otherwise constrain a provisional literal, it keeps the narrowest concrete type that represents that literal.

## Automatic Conversion

First automatically converts compatible numeric representations:

1. A provably lossless conversion is inserted silently.
2. A potentially lossy conversion is inserted and receives the saw decoration (🪚).
3. `declare noAutomaticNumberConversion` reports potentially lossy automatic conversions within its scope. An explicit conversion satisfies the declaration.

The saw is an editor decoration. It communicates that the generated program contains a conversion that can discard information.

```first
source is u16 = 257
target is u8 = 🪚source

console.log(target) // 1
```

First still produces compiled output when an automatic conversion violates `declare noAutomaticNumberConversion`. The compiler reports a notice and uses the ordinary conversion as its recovery behavior.

## Fixed-Decimal Conversion

Converting a fixed decimal to a smaller scale truncates toward zero by default.

```first
source is i32.4 = 1.2399
target is i32.2 = 🪚source

console.log(target) // 1.23
```

Potentially discarding fractional digits receives a saw. Rounding and round-and-clamp behavior are requested through explicit conversion operations rather than a scoped rounding declaration.

## Integer-To-Float Conversion

An integer type converts losslessly to a floating-point type only when every value of the source type is exactly representable in the target type.

| Source | Lossless targets |
| --- | --- |
| `i8`, `u8` | `f16`, `f32`, `f64`, `f128` |
| `i16`, `u16` | `f32`, `f64`, `f128` |
| `i32`, `u32` | `f64`, `f128` |
| `i64`, `u64` | `f128` |
| `i128`, `u128` | None |

There is no type-wide provably lossless conversion from floats to integers, floats to fixed decimals, fixed decimals to floats, or arbitrary-precision numbers to fixed-width numbers.

## Type Attestations

`numeric` can test whether an otherwise broad value is one of First's built-in numeric primitives. `number` tests specifically for `f64` because it is an alias for that type.

```first
write(value is any) (
	if (value is numeric) (
		console.log(value)
	)
)
```

A provisional value may also be tested against its currently selected concrete primitive. The result is determined statically for a given compilation.

```first
x is var numeric = 0

if (x is u8) (
	console.log("x currently resolves to u8")
)
```

Editing the surrounding program may cause the compiler to select another concrete type and therefore change the result of this attestation in the next compilation.
