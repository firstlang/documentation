# Strings

First has two string primitives:

- `string` contains Unicode text.
- `byte string` contains bytes.

Both are immutable value types. Assignment, parameter passing, and return use value semantics even when the compiler shares, borrows, moves, or interns their storage. Neither type has observable object identity, and neither can use `weak` ownership.

First does not have a capitalized `String` object type. `string` is the ordinary string type.

## Unicode Model

An ordinary `string` is a sequence of Unicode scalar values. Unicode scalar values exclude the UTF-16 surrogate range, so an ordinary string cannot contain an isolated surrogate.

First distinguishes three measurements of text:

- A _byte_ is an integer from `0` through `255`.
- A _code point_ in an ordinary string is a Unicode scalar value.
- A _grapheme_ is an extended grapheme cluster: the closest Unicode approximation of one user-perceived character.

One grapheme may contain more than one code point. For example, the family emoji contains seven code points joined into one grapheme.

The `char` type represents one code point, as defined in [[05-Chars]]. It does not represent an entire multi-code-point grapheme.

## Ordinary String Literals

String literals have two delimiter forms: double quotes contain noninterpolating text, and backticks allow `{expression}` interpolation. Both forms support physical newlines and the indentation rules below. Single quotes are reserved for `char` literals.

```first
name is string = "First"
letter is char = 'T'
```

A plain untagged string literal is context-sensitive compile-time information. Its family is selected as `string` or `byte string` before its contents and interpolations are converted. When no context pins it to `byte string`, it becomes `string`.

```first
text = "hello" // string
bytes is byte string = "hello" // byte string
```

This contextual selection does not create a runtime provisional-string type.

After a literal is selected as `byte string`, its source characters, escapes, and interpolations use the byte-literal rules below.

### Escapes

Ordinary literals support TypeScript's string escape forms, including:

```first
quote = "\""
backslash = "\\"
controls = "\0\b\f\n\r\t\v"
hexadecimal = "\x41"
unicodeUnit = "\u0041"
unicodeScalar = "\u{1F600}"
```

First does not support legacy octal string escapes or backslash-line continuation. Physical newlines use the multiline rules below. Both delimiter forms support escapes; noninterpolating strings are not raw strings. Escape the enclosing delimiter with a backslash when it occurs as text.

An escape must ultimately produce valid Unicode scalar values. An escape outside the scalar range, including an isolated UTF-16 surrogate, is not a valid First string value. The compiler reports a notice and substitutes `U+FFFD`, the Unicode replacement character, as recovery.

Adjacent fixed-width Unicode escapes that form a valid UTF-16 surrogate pair are combined into their one represented scalar before isolated-surrogate recovery is applied.

```first
emoji = "\uD83D\uDE00" // "😀"
```

```first
value = "\uD800" // ❌ Wrong: isolated surrogate; recovers as U+FFFD
```

An invalid escape retains its source characters literally and produces a notice. The recovery does not silently discard the backslash or the characters following it.

An unescaped non-line-terminator control character that cannot normally appear in a string literal is retained and produces a notice.

Source files may contain Unicode characters directly. Authors do not need to escape valid Unicode text.

### Physical Lines

Both double-quoted and backtick literals may span physical lines. The following rules apply to both `string` and `byte string`:

1. Normalize physical CRLF and CR line endings to LF. Explicit escapes such as `\r\n` retain their specified values.
2. Exclude the first and last lines when they contain only ASCII spaces or tabs, regardless of their indentation. Remove each excluded boundary line together with its adjacent line break, without removing the same line break twice. This applies to multiline literals, not single-line whitespace strings.
3. Find the longest identical leading sequence of ASCII spaces and tabs shared by all remaining nonblank lines. Include content on the opening-delimiter line. A nonblank line with no leading whitespace makes this prefix empty. Whitespace-only lines do not determine the prefix; if no nonblank lines remain, the prefix is empty.
4. Remove that prefix from every nonblank line. From whitespace-only lines, remove the matching portion of the prefix, stopping at the first mismatch or the end of the line. Preserve interior line breaks.

Tabs are not expanded to columns. Mixed tabs and spaces are meaningful, for example when embedding YAML. Only whitespace inside the literal participates; indentation before its opening delimiter is not part of its content.

Trimming operates on literal source text before escape decoding and interpolation evaluation. An interpolation island is an opaque content unit for outer indentation measurement; physical lines inside its expression do not participate. Inserted values are unchanged and receive no automatic indentation. Nested string literals apply their own indentation rules independently.

```first
message = "
	first line
	second line
"
// "first line\nsecond line"

inline = "first line
	second line"
// "first line\n\tsecond line": the first line has no indent.

value = "a\n\tb"
inserted = `
	{value}
`
// "a\n\tb": inserted whitespace is preserved.
```

The compiler may fold constant concatenations. 

## Interpolation

Only backtick string literals support interpolation. An interpolation begins with `{` and ends with the structurally matching `}`. Double-quoted literals preserve braces as ordinary text.

```first
name = "First"
message = `Hello, {name}`
```

Spaces around the expression are optional and have no semantic effect.

```first
compact = `{value}`
spaced = `{ value }`
```

Interpolation expressions use the ordinary expression grammar. They may contain nested parentheses, object braces, and string literals with their own interpolations. Braces inside nested constructs do not close the enclosing island.

```first
message = `Result: {format((left + right), `nested {value}`)}`
```

Escape an opening brace as `\{` to include it as text in a backtick literal. A closing brace outside an island is ordinary text; `\}` is also accepted as an escaped delimiter.

```first
literal = `\{value}` // "{value}"
plain = "{value}" // "{value}", without interpolation
```

Escaped delimiters are recognized before interpolation parsing.

Interpolation expressions evaluate from left to right. Each expression evaluates exactly once.

An empty interpolation inserts no text and produces a notice.

```first
value = `before{ }after` // "beforeafter", notice
```

Ordinary interpolation always produces `string`. Its result cannot be `byte string`, even when every resulting character is ASCII.

### Interpolation Conversion

An interpolated value is converted by invoking the `toString()` selected through ordinary member lookup. The selected member must:

- Be accessible at the interpolation site.
- Be callable without arguments and therefore have no required parameters.
- Resolve without an overload ambiguity.
- Return `string`.

Optional parameters do not prevent implicit invocation because the function remains callable without arguments.

An inaccessible, ambiguous, incorrectly typed, missing, or argument-requiring `toString()` inserts no text and produces a notice. This recovery does not prevent later interpolation expressions from evaluating.

Classes and custom primitive classes do not receive JavaScript's `"[object Object]"` fallback. Their values must provide a qualifying `toString()` when they are interpolated.

Type values are handled directly by the compiler. Interpolating a class, primitive class, or selection type itself produces its declared type name rather than requiring runtime reflection.

```first
`{Animal}` // "Animal"
`{Bunny}` // "Bunny"
`{Day}` // "Day"
```

The declared name is the identifier on the type declaration. Generic arguments and containing-space or containing-class qualification are not appended. This rule concerns the type itself, not an instance or selected value belonging to that type.

### Built-In Stringification

First defines the built-in stringification of each supported primitive explicitly.

```first
`{true}` // "true"
`{false}` // "false"
`{null}` // "null"
`{nan}` // "nan"
`{infinity}` // "infinity"
`{-infinity}` // "-infinity"
```

Integers use ordinary base-10 notation. Fixed decimals preserve exactly the number of fractional digits declared by their scale. Floating-point values use their canonical shortest decimal representation unless an explicit formatting function is called. Built-in numeric conversion does not insert locale-specific grouping or decimal separators.

Negative floating-point zero stringifies as `"0"`.

Interpolating a `char` inserts its scalar value as text. Interpolating a `string` inserts it unchanged.

Interpolating a `byte string` invokes its built-in `toString()`, which uses [Byte String To String Conversion](#byte-string-to-string-conversion). The interpolation receives a saw when the byte value may contain bytes above `127`.

Arrays use JavaScript-compatible comma-joined string conversion. Elements recursively use their own string conversion.

```first
`{ [1, 2, 3] }` // "1,2,3"
`{ [[1, 2], [3, 4]] }` // "1,2,3,4"
`{ [] }` // ""
```

An array's `null` elements contribute empty fields, matching JavaScript array conversion. If an element lacks a qualifying `toString()`, that element likewise contributes an empty field and produces a notice; commas belonging to the array remain.

```first
`{ [1, null, 3] }` // "1,,3"
```

A value of a selection type stringifies through its selected underlying value. Interpolating the selection type itself instead produces the type's declared name.

Interpolation has no width, precision, radix, padding, debug, or other format-specifier syntax. Authors call the corresponding formatting function inside the interpolation expression.

Interpolation itself adds no locale behavior. A called formatting function or user-defined `toString()` retains whatever locale behavior that function declares.

## Length

The `length` property of `string` returns `u64` and counts extended grapheme clusters.

```first
plain = "hello"
emoji = "😀"
family = "👨‍👩‍👧‍👦"
precomposed = "é"
decomposed = "é"

plain.length // 5
emoji.length // 1
family.length // 1
precomposed.length // 1
decomposed.length // 1
```

The `codeLength` property returns `u64` and counts Unicode scalar values. It is the upper bound used by scalar indexing and ordinary string iteration.

```first
family.codeLength // 7
decomposed.codeLength // 2
```

Grapheme boundaries follow the Unicode grapheme-cluster algorithm pinned by the compiler and standard library version. First's `Intl.Segmenter` uses the same pinned Unicode data. A backend cannot substitute host segmentation data when that data would produce different boundaries. Updating the pinned Unicode version is a documented compatibility change.

`codeLength` is O(1). Computing `length` may require grapheme segmentation. The compiler or runtime may cache grapheme boundaries because strings are immutable, but the language does not guarantee that the first `length` access is O(1).

Encoding-specific measurements, including UTF-8 byte length and UTF-16 code-unit length, belong to the corresponding encoding APIs. They are not ordinary string positions.

Unless an API explicitly names bytes, UTF-16 code units, or graphemes, every ordinary-string position it consumes or returns is a Unicode scalar position. This includes search positions, match offsets, and the positional parameters of ECMAScript-derived string methods.

## Scalar Indexing

Bracket indexing uses zero-based Unicode scalar positions and returns `char`.

```first
text = "café"

text[0] // 'c'
text[3] // 'é'
```

Bracket indexes are bounded by `codeLength`, not by the grapheme-counting `length` property.

```first
family = "👨‍👩‍👧‍👦"

family.length // 1
family.codeLength // 7
family[0] // '👨'
family[1] // U+200D ZERO WIDTH JOINER
family[6] // '👦'
```

Scalar indexing is O(1). Generated representations must provide direct scalar lookup even when they also use a variable-width encoding internally.

### Index Recovery

String indexing is total. First does not have `undefined`, and making every indexed access nullable would add null handling to every ordinary character lookup.

For a nonempty string:

- A negative index clamps to position `0`.
- An index at least `codeLength` clamps to position `codeLength - 1`.

Indexing an empty string returns the NUL character `\0`.

```first
"abc"[-1] // 'a'
"abc"[100] // 'c'
""[0] // '\0'
```

When the compiler can prove from constant values that an access will clamp or use the empty-string fallback, it produces a compiler notice at that access. When the condition depends on runtime values, the debugging interpreter reports the notice if fallback behavior occurs. Compiled execution always uses the same deterministic value and does not change control flow.

`codePointAt(index)` uses the same scalar positions and recovery rules as bracket indexing. It returns the selected scalar's numeric value as `u32`; the empty-string fallback therefore returns `0`.

```first
"A".codePointAt(0) // 65
"".codePointAt(0) // 0
```

An index may use any concrete fixed-width or arbitrary-precision integer type. Its mathematical value is compared with the available positions before backend index conversion, so a value wider than the backend's native index type clamps correctly instead of wrapping. Provisional numeric indexes are pinned to an integer representation. A non-integer numeric index uses the corresponding ordinary numeric-to-integer conversion, truncates toward zero, and receives a saw before string-index recovery is applied. An index expression whose type cannot convert to an integer produces a notice and recovers as index `0`.

## String Iteration

Ordinary `string` iteration visits Unicode scalar values in sequence order and produces one `char` for each value. It uses the same sequence and positions as bracket indexing and `codeLength`.

Iteration does not combine scalars into graphemes. Iterating the family emoji visits each person and each zero-width joiner separately. This keeps ordinary iteration compatible with `char` and permits a direct scalar iterator.

Ordinary iteration syntax is defined in [[04-Loops]]. Numeric traversal can use the scalar positions from `0` up to but excluding `codeLength`; range construction and iteration are defined in [[08-Ranges]] and [[04-Loops]].

Grapheme-, word-, and sentence-aware traversal uses the standard `Intl.Segmenter` API. First retains that ECMA-402 surface for compatibility instead of adding separate `graphemes()` or `graphemeAt()` string members.

## String Slicing

Ordinary string slicing uses Unicode scalar positions. A slice may separate code points that previously belonged to one grapheme, but it never divides a scalar value.

Slicing methods derived from ECMAScript retain their method-specific boundary rules after replacing UTF-16 code-unit positions with First scalar positions. For example, `slice()` interprets negative positions relative to `codeLength`, clamps positions to the available scalar sequence, and returns an empty string when its resulting interval is empty. Boundary adjustment defined by a slicing method is ordinary behavior and does not produce the invalid-index notice used by bracket access.

Slicing an ordinary string returns `string`. The generated representation may share storage with the source, but ownership and storage sharing are not observable through string behavior.

## Concatenation

The `+` operator concatenates string and character values. It evaluates its left operand before its right operand and evaluates each operand exactly once.

The following operations produce `string`:

```first
"left" + "right"
"value: " + 'A'
'A' + " value"
```

Two `char` values produce `byte string` when both are statically known to be ASCII. Otherwise they produce `string`.

```first
'a' + 'b' // byte string containing "ab"
'é' + '😀' // string containing "é😀"
```

Concatenating `string` with `byte string` produces `string`. The byte operand first undergoes the saw-bearing conversion defined in [Byte String To String Conversion](#byte-string-to-string-conversion).

The byte-preserving concatenation cases are defined under [Byte String Concatenation](#byte-string-concatenation).

Numbers, booleans, objects, and other non-string values do not acquire implicit `+` conversion merely because they have a `toString()` member. Use interpolation or call `toString()` explicitly.

```first
"Count: " + 10 // ❌ Wrong: use interpolation or toString()
```

Strings are immutable. `+=` creates a new value and reassigns a mutable binding; it does not modify existing string storage.

```first
text is var string = "a"
text += "b"
```

The compiler may fold constant concatenations and combine the allocations of a concatenation chain. Those optimizations do not alter evaluation order.

## Byte Strings

`byte string` is an immutable sequence of `u8` values. It has no inherent text encoding. The name describes its representation and positional behavior, not a guarantee that its contents are textual ASCII.

```first
header is byte string = byte "HTTP"
binary is byte string = byte "\x00\x80\xFF"
```

Bytes from `0` through `127` have their ordinary ASCII meanings. Bytes from `128` through `255` retain stable numeric identity. They can be widened numerically to `char`, while automatic whole-value conversion to `string` deliberately maps them to NUL as defined below.

### Byte String Literals

Tagged byte literals place a space between `byte` and the opening delimiter. The `byte` prefix may be applied to either string form, so both the double-quoted form `byte "..."` and a backtick-delimited literal produce `byte string`. The only difference is interpolation: the backtick form supports interpolation, while the double-quoted form does not.

```first
plain = byte "hello"
greeting = byte `Hello, {name}`
```

A tagged byte literal has the concrete type `byte string`. It is not context-sensitive.

As established for plain literals above, an untagged literal may instead become `byte string` when required by context.

ASCII source characters and ordinary escapes whose values are ASCII contribute their corresponding byte values. `\xNN` contributes exactly one byte and is the canonical way to write an arbitrary byte.

```first
controls = byte "\0\n\r\t"
binary = byte "\x00\x7F\x80\xFF"
quote = byte "\""
backslash = byte "\\"
```

An explicit `\xNN` byte never receives a saw merely because its value exceeds `127`; the author requested that byte directly.

An invalid byte escape retains its source characters as bytes and produces a notice, following the ordinary invalid-escape recovery. Any retained non-ASCII source character then undergoes the ordinary saw-bearing NUL conversion.

A direct non-ASCII character or a Unicode escape outside the ASCII range converts to one NUL byte. The conversion receives a saw because it discards the original scalar. It does not additionally produce a notice.

```first
value = 🪚byte "café" // Bytes 63 61 66 00
escaped = 🪚byte "\u00E9" // One 00 byte
```

An escape denoting an isolated surrogate is malformed before byte conversion. It receives the ordinary malformed-Unicode notice, recovers as `U+FFFD`, and that recovered non-ASCII scalar then becomes NUL with a saw.

A valid surrogate-pair escape is combined into one scalar first and therefore contributes one NUL byte with a saw.

The saw is an editor decoration listed in [[08-Glyphs-And-Editor-Expansions]]. It identifies possible information loss; it is not an error or a notice.

Both byte-literal delimiter forms support physical newlines and use the same boundary-line removal, common-prefix trimming, and physical line-ending normalization as ordinary strings. Explicit byte escapes retain their values.

### Byte String Interpolation

Backtick byte literals support the same `{expression}` interpolation grammar, nesting, escaping, evaluation order, and empty-interpolation recovery as ordinary backtick string literals. Double-quoted byte literals do not interpolate.

```first
name is byte string = byte "First"
message = byte `Hello, {name}`
```

Byte interpolation always produces `byte string`.

An interpolated `byte string` contributes its bytes directly. Every other value first undergoes ordinary interpolation stringification and then the saw-bearing `string` to `byte string` conversion. The saw appears because the inserted value is not already byte-typed and may lose non-ASCII scalars.

```first
name is string = "Renée"
message = byte `Hello, {🪚name}`
```

If stringification cannot find a qualifying `toString()`, the interpolation inserts no bytes and produces the ordinary interpolation notice.

### Byte Length And Indexing

The `length` property of `byte string` returns `u64`, counts stored bytes, and is O(1).

```first
bytes = byte "A\xFFB"

bytes.length // 3
```

Bracket indexing uses zero-based byte positions and returns `u8` in O(1).

```first
bytes[0] // 65, as u8
bytes[1] // 255, as u8
```

Byte indexes use the same integer-conversion and invalid-index-type recovery rules as ordinary string indexes.

Byte indexing uses the same total boundary policy as ordinary string indexing:

- A negative index clamps to position `0`.
- An index at least `length` clamps to position `length - 1`.
- Indexing an empty byte string returns `0` as `u8`.

A statically provable fallback produces a compiler notice. A fallback discovered during interpreted execution produces a debugging-interpreter notice. Compiled execution returns the deterministic fallback without changing control flow.

### Byte String Iteration

Byte-string iteration advances one stored byte at a time. It supports a two-value iteration binding:

```first
text is byte string = "hello"

text each code character (
	console.log(code)
	console.log(character)
)
```

The first binding, `code`, is the byte as `u8`. The second binding, `character`, is optional and contains the `char` obtained by numerically widening that byte to the scalar from `U+0000` through `U+00FF`.

`code` and `character` are ordinary author-chosen binding names, not fixed keywords.

When the second binding is omitted, the compiler does not construct a `char` for the iteration.

```first
text each code (
	console.log(code)
)
```

This iteration is a naïve increment through byte storage. It does not perform UTF-8 decoding or grapheme segmentation. Its binding rules are also owned by [[04-Loops]].

### Byte String Slicing

Byte-string slicing uses byte positions and returns `byte string`. It never decodes or rewrites the selected bytes.

Each byte-slicing operation defines its boundaries in byte positions. Its ordinary boundary adjustment does not produce an invalid-index notice. Slicing may share storage internally, but the result remains an immutable value.

### Byte String Concatenation

Concatenating two byte strings produces `byte string` and copies their byte sequences without text conversion.

```first
byte "ab" + byte "cd" // byte string containing 61 62 63 64
```

A byte string and a `char` produce `byte string` without a saw when the character is statically known to be ASCII.

```first
byte "ab" + 'c' // byte string
'a' + byte "bc" // byte string
```

When the character is not statically known to be ASCII, concatenation widens to `string`. The byte-string operand undergoes byte-to-string conversion; the `char` remains its ordinary Unicode scalar.

The byte-to-string conversion receives a saw when the byte operand may contain a value above `127`.

When concatenation produces `byte string`, `+=` creates a new byte sequence and reassigns a mutable byte-string binding. Concatenation that widens to `string` follows the ordinary assignment-conversion rules instead. The compiler may coalesce allocations without changing evaluation order.

## Conversion Between String Families

Automatic conversion between `string` and `byte string` is positional rather than an implicit character-encoding operation. UTF-8 and other encoded transformations belong to the standard Web Encoding APIs.

### String To Byte String Conversion

Converting `string` to `byte string` processes one Unicode scalar at a time:

- Scalars from `U+0000` through `U+007F` become the equal-valued byte.
- Every scalar above `U+007F` becomes the NUL byte `0`.

The conversion preserves scalar-to-byte index correspondence: the result's byte `length` equals the source's `codeLength`. It receives a saw wherever information loss is possible.

```first
source is string = "café"
bytes is byte string = 🪚source // Bytes 63 61 66 00
```

This conversion is not UTF-8 encoding. Use the explicit Web Encoding APIs when encoded text must preserve non-ASCII content.

### Byte String To String Conversion

Converting `byte string` to `string` processes one byte at a time:

- Bytes from `0` through `127` become the equal-valued Unicode scalar.
- Bytes from `128` through `255` become the NUL scalar `U+0000`.

The conversion preserves byte-to-scalar index correspondence: the result's `codeLength` equals the source's byte `length`. It receives a saw wherever a byte above `127` may occur.

```first
source is byte string = byte "A\xFFB"
text is string = 🪚source // "A\0B"
```

The conversion does not attempt UTF-8, Latin-1, transliteration, or replacement-character decoding. It is not guaranteed to round-trip bytes outside the ASCII range.

The optional `character` binding in byte iteration is different: it explicitly widens each `u8` numerically to `char`, so byte `255` becomes `U+00FF`. Whole-value conversion to `string` instead maps that byte to NUL.

The built-in `byte string.toString()` uses this whole-value byte-to-string conversion. A byte-oriented API may instead accept `byte string` directly and expose its raw values according to that API; it does not obtain raw bytes by implicitly treating them as ordinary Unicode text.

## Equality And Ordering

Two ordinary strings are equal when they contain the same scalar sequence. Two byte strings are equal when they contain the same byte sequence.

For two values of the same string family, `==` and `===` produce the same result because neither primitive has object identity or a boxed-object form.

First never normalizes Unicode implicitly during literal construction, concatenation, interpolation, slicing, or comparison. Equality also does not automatically convert between `string` and `byte string`.

```first
"é" == "é" // false
byte "abc" == byte "abc" // true
```

An equality or relational operation directly mixing `string` and `byte string` produces a notice and recovers as `false`. Authors must choose an explicit conversion when cross-family comparison is intentional.

The first comparison is false because the first value contains `U+00E9`, while the second contains `U+0065` followed by `U+0301`. They render alike and each has grapheme `length` 1, but their scalar sequences differ. Authors may explicitly normalize strings through the standard `normalize()` member before comparison.

Relational operators compare ordinary strings lexicographically by scalar value and byte strings lexicographically by byte value. These comparisons are naïve and deterministic. They do not perform locale collation.

```first
"a" < "b" // true
byte "a" < byte "b" // true
```

Locale-sensitive comparison uses the explicit ECMA-402 collation APIs.

## Explicit Boolean Conversion

Strings do not participate in implicit truthiness. A condition must be boolean as defined in the control-flow specification.

Explicit Boolean conversion produces `false` for an empty `string` or empty `byte string` and `true` for a nonempty value.

## Locale Behavior

Ordinary equality, ordering, indexing, length, concatenation, and built-in primitive stringification do not consult the machine's ambient locale. Locale-aware formatting, collation, casing, word segmentation, and related operations are explicit ECMA-402 standard-library behavior.

Ordinary `toLowerCase()` and `toUpperCase()` use Unicode default casing. Locale-aware variants such as `toLocaleLowerCase()`, `toLocaleUpperCase()`, and `localeCompare()` use the locale explicitly supplied through their standard-library APIs.

A future declarative-locale invariant will use the canonical declaration form:

```first
declare locale frCA
```

Its scope, inheritance, override rules, and runtime locale-data requirements are future work and are not defined by this document.

## Representation And Lowering

The observable semantics do not require one Rust representation.

An ordinary string may lower to scalar storage, UTF-8 plus a scalar-offset table, inline storage, shared immutable storage, or another representation that preserves:

- Valid Unicode scalar contents.
- O(1) `codeLength`.
- O(1) scalar bracket indexing and `codePointAt()`.
- The specified grapheme `length`.

A byte string may lower directly to Rust byte storage such as a byte slice or byte vector. Borrowing, moving, cloning, copy-on-write storage, and reference-counted storage are compiler decisions and do not change First value semantics.

First string operations must not lower naïvely to JavaScript's UTF-16 `length` or indexing operations in a future JavaScript backend. Such a backend may use generated runtime helpers or indexed representations while still using TypeScript as a type-checking intermediate.

## Interning And Constant Folding

Literal strings are interned by default when doing so preserves their value semantics. Constant literal concatenation and constant interpolation may be folded.

Interning is not observable through program-level equality or identity because strings have no object identity. Typist may expose interning, shared storage, encoding, and other lowering choices through an implementation-focused editor view.

A future invariant declaration may allow authors to require or prohibit particular interning behavior. That invariant is not defined here.

## Diagnostic Recovery

Notices identify malformed syntax, unavailable stringification, and index fallback at the points defined above. Saws identify the potentially lossy conversions defined above and are separate from notices. These notice- and saw-bearing recovery rules do not interrupt execution or change control flow.
