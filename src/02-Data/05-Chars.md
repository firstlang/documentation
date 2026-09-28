# Characters

`char` represents one Unicode scalar value. Its validity, range, and representation match Rust's `char` type.

A Unicode scalar value is any Unicode code point from `U+0000` through `U+10FFFF`, excluding the surrogate range `U+D800` through `U+DFFF`.

```first
letter is char = 'A'
accented is char = 'é'
emoji is char = '😀'
```

First has one character type. It does not provide separate `char8`, `char16`, or `char32` primitives. A `char` is stored in 32 bits regardless of the number of bytes required to encode it as UTF-8.

## Scalars And Displayed Characters

A `char` is one Unicode scalar value, not necessarily one user-perceived character. A displayed character may contain multiple scalars, such as a base letter followed by a combining mark.

```first
'é'	// One scalar: U+00E9
'é'	// ❌ Wrong: two scalars in one character literal
```

Character literals contain exactly one scalar value. The editor reports a notice when a character literal contains no scalar values or more than one scalar value.

## Strings

Ordinary `string` values contain valid Unicode scalar sequences. Encoding an ordinary string as UTF-8 uses between one and four bytes for each `char`, even though the `char` value itself always occupies 32 bits.

Ordinary-string indexing and iteration produce `char` values. Encoding, byte-string indexing, grapheme segmentation, string slicing, and out-of-range behavior are defined separately in [[06-Strings]].

## Integer Conversion

Converting a `char` to `u32` returns its Unicode scalar value and is lossless.

```first
codePoint is u32 = 'A' // 65
```

Conversions to narrower integers use the ordinary numeric conversion rules and receive a saw when they may lose information.

Integer conversion exposes a character's scalar value. It does not encode the character as UTF-8, UTF-16, or another character encoding. Encoding is performed through the corresponding encoding APIs.

## Numeric Groups

`char` belongs to `AnyNumeric` and `AnyInteger`. This permits character-aware constraints and ranges without defining a separate `AnyChar` group.

Group membership does not give `char` arithmetic operators. Convert a character to an integer before performing numeric arithmetic.

```first
'A' + 1 // ❌ Wrong: char does not support arithmetic

codePoint is u32 = 'A'
nextCodePoint = codePoint + 1
```
