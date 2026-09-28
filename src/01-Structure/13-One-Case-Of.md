The `one case of` feature is a selection type for choosing exactly one named case.

It is First's spelling for algebraic-data-type-style values. Each case has its own constructor shape, and each constructed value carries exactly one active case.

The shared selection type rules are defined in [[Selection Types]].

## Basic Syntax

Every case is written with function-call syntax.

```first
Message is one case of (
	default()
	text(value is string)
	image(url is Url)
	reaction(emoji is string)
)
```

No-payload cases use an empty parameter list.

```first
default()
```

Payload-bearing cases declare one or more parameters.

```first
text(value is string)
image(url is Url)
```

The declaration has no implementation body. The case list defines the constructors and the possible payload shapes.

## Construction

A `one case of` declaration creates an access surface with one constructor per case.

```first
message is Message = Message.default()
message = Message.text("hello")
message = Message.image(Url("http://example.com"))
message = Message.reaction(":)")
```

Payloadless cases are still called with `()`.

```first
Message.default()
```

This keeps all cases conceptually uniform: every case is a constructor, and some constructors take no values.

## Cases And Payloads

Each case creates a distinct shape inside the selection type.

```first
Result is one case of (
	ok(value is string)
	failed(code is u16, message is string)
)
```

The active case determines which payload fields exist.

The payload fields are not ordinary properties on the whole selection type. They are available only after the value has been narrowed to the corresponding case.

## Matching

Programs inspect a `one case of` value by matching on its active case.

```first
handle(message is Message) (
	message matches (
		default() console.log("default")
		
		text(🥚 value) console.log(value)
		
		image(🥚 url) console.log(url)
		
		reaction(🥚 emoji) console.log(emoji)
	)
)
```

A match arm names the case and may declare local bindings for the payload values declared by that case. In the editor, new payload bindings are rendered with the `🥚` decoration.

```first
text(🥚 value) console.log(value)
```

An ordinary identifier does not create a new local binding. It must resolve to a statically resolvable compile-time constant under the ordinary match-pattern rules.

```first
handle(message is Message) (
	🥚 expected = "the-value"
	
	message matches (
		text(expected) console.log("matched the expected value")
	)
)
```

When a payload value must match a specific literal or constant instead of being bound as a local, the arm writes that value directly.

```first
message matches (
	reaction(":)") console.log("smile")
)
```

Case patterns may omit trailing payload positions when those values are not needed.

```first
message matches (
	text() handleAnyText()
)
```

This omission is available only in patterns. Calling `Message.text()` as a constructor still requires its declared payload.

`always matches` opts into complete case coverage. The compiler emits a prominent notice and injects an `else null` recovery when any case remains uncovered.

```first
message always matches (
	default() showDefault()
	text() showText()
	image() showImage()
	reaction() showReaction()
)
```

The complete ordering, binding, shared-body, result, and exhaustiveness rules are defined in [[03-Matches]].

## Structural Typing

`one case of` selection types are structurally typed like other selection types.

Two `one case of` declarations are compatible when they describe the same case names and compatible payload shapes.

```first
Message is one case of (
	text(value is string)
	image(url is Url)
)

Notification is one case of (
	text(value is string)
	image(url is Url)
)
```

`Message` and `Notification` describe the same case structure.

## Names

Case names must be unique within a `one case of` declaration.

```first
Message is one case of (
	text(value is string)
	text(value is string) // notice
)
```

Duplicate case names generate a compiler notice.

Repeated case names are conflicts even when they have identical payloads or arrive from the same ancestor through a diamond. Report a notice and retain the later case as recovery. Do not merge payloads, introduce overloads, or require qualified disambiguation.

## Composition

Compose case selections in the header with `or`. Body spreads are not supported.

```first
CommonMessage is one case of (
	default()
	text(value is string)
)

Message is CommonMessage or one case of (
	image(url is Url)
	reaction(emoji is string)
)

Combined is First or Second or Third
```

Composition preserves source order. Duplicate case names always produce notices, including diamond inheritance and identical declarations. The later case wins recovery. The original source declarations remain unchanged.

## Methods And Extensions

`one case of` selection types may define functions.

They may also be extended with extension classes.

They do not support properties, fields, constructors, nested spaces, or ghosts.

## Lowering

A `one case of` value lowers to a tagged representation.

The selected case determines the tag. Payload-bearing cases store their payload values beside that tag.

For Rust output, `one case of` naturally lowers to an enum with one variant per case.

```rust
enum Message {
	Default,
	Text(String),
	Image(Url),
	Reaction(String),
}
```

The generated backend representation is an implementation detail. First source should use case constructors and matching rather than depending on a specific tag layout.
