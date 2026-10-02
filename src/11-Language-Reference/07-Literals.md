# Literals

This page lists First's literal forms. It is a syntactic reference, not the full semantic definition of each value family. Detailed behavior belongs to the linked topic pages.

## Primitive Literals

| Form | Kind | Notes |
| ---- | ---- | ----- |
| `12` | Provisional integer number | The concrete integer type is selected by context and later type flow. See [[04-Provisional-Numbers]]. |
| `-12` | Negative provisional integer number | The sign is part of the literal form where a literal value is required. |
| `12u` | Provisional unsigned integer number | The concrete unsigned type is selected by context and later type flow. |
| `12.34` | Provisional fixed-decimal number | The concrete fixed-decimal type is selected by context and later type flow. |
| `12.34f` | `f32` number | Explicit binary floating-point literal. |
| `12.34ff` | `f64` number | Explicit binary floating-point literal. |
| `12.34fff` | `f128` number | Explicit binary floating-point literal. |
| `0x12AB` | Hex provisional integer number | Hex digits may use uppercase or lowercase letters. |
| `12px` | Quantity literal | The unit name is user-defined. |
| `12.5kg` | Quantity literal | Decimal quantities use the same user-defined unit namespace. |
| `true` | Boolean literal | The true `boolean` value. |
| `false` | Boolean literal | The false `boolean` value. |
| `null` | Null literal | The only value of the `null` type. |
| `nan` | Floating-point special value | Lowercase spelling. |
| `infinity` | Floating-point special value | Lowercase spelling. |
| `-infinity` | Floating-point special value | Negative infinity. |
| `'A'` | Character literal | Contains exactly one Unicode scalar value. See [[05-Chars]]. |

## Text Literals

| Form | Kind | Notes |
| ---- | ---- | ----- |
| `"text"` | String literal | Noninterpolating text. See [[06-Strings]]. |
| `` `text {value}` `` | Interpolated string literal | Supports `{expression}` interpolation. |
| `byte "text"` | Byte string literal | Noninterpolating byte string. |
| `` byte `text {value}` `` | Interpolated byte string literal | Supports `{expression}` interpolation and produces `byte string`. |

String and byte string literals may span physical lines. Their escaping, indentation, interpolation, and byte conversion rules are defined in [[06-Strings]].

## Regular Expression Literals

Regular expression literals follow the current JavaScript regular expression literal form.

```first
pattern = /[a-z0-9]+/i
```

Their pattern grammar, flags, escaping, and runtime behavior are the JavaScript regular expression rules supported by the target environment.

## Aggregate Literals

| Form | Kind | Notes |
| ---- | ---- | ----- |
| `[1, 2, 3]` | Array literal | Entries are ordinary expressions. See [[09-Arrays]]. |
| `{ name is string = "bob" }` | Object literal | Fields use First field syntax. See [[10-Objects]]. |

Array and object literals are also permitted as exact literal values in selection entries and match patterns when the surrounding feature allows literal alternatives.

## Range Literals

Ranges are created with range literal syntax.

```first
0 to 10
0 til 10
0 to 10 step 2
```

`to` creates an inclusive range. `til` creates an exclusive range. The optional `step` expression controls spacing. See [[08-Ranges]].

## Markup Literals

Markup literals use First's JSX-like surface without a separate file extension.

```first
link = <a class=name href=url>Text {value}</a>
```

Attribute values are ordinary expressions. Text may contain `{expression}` islands. Markup construction is controlled by the active markup factory declaration. See [[11-Markup-Literals]].

## Resource Literals

Resource tokens have the form `scheme:payload`.

```first
figma:file-id
asset:icons/logo.svg
```

They are not general expression literals. Resource tokens are only permitted inside anchor islands, where the scheme and payload are preserved for tools or implementations that understand them.
