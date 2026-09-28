
First supports Markup Literals, but these have some minor differences from JSX. These differences are designed to make the syntax feel more First-native. JSX/TSX feel like a different language bolted on JavaScript and TypeScript, and we aim to avoid that.

First doesn't use a separate file extension to support markup literals like TypeScript. The parser ambiguity in TypeScript comes from `<Type>` casts not greater than / less than. We don't have those in First so we can actually support markup literals without much additional effort.

Valid JSX looks like this:

```first
<a className={value} href={value}>text before {someVariable} text after</a>
```

But the First syntax looks like this:

```first
<a class=value href=value>text before {someVariable} text after</a>
// Or with parens
<a class=(a + b) href=(a + b)>text before someVariable text after</a>
```

Functions are just like this:

```first
<a onclick=( () => console.log("clicked") )>Link</a>
```

Or they could be like this, because the ( ) syntax in First creates a statement list.

```first
<a onclick=( console.log("clicked") )>Link</a>
```

There is potential for a parsing ambiguity. In tokens, we simply wrap all the values in ( and ) characters to avoid this.

```first
// Would be ambiguous
<a onclick=() => 1 > 2 || 3 > 4</a>

// Wrapped in parenthesis, the ambiguity no longer stays
<a onclick=( () => 1 > 2 || 3 )> 4</a>
```

Attribute values remain ordinary expressions, while embedded text uses `{expression}` islands, sharing the delimiters used by backtick strings and anchors. Nested expression constructs do not close the enclosing island. In text, `\{` writes a literal opening brace; a closing brace outside an island is literal text. Quoted attribute strings follow [[06-Strings]]: double quotes do not interpolate, and backticks do. The string indentation rules apply to string literals, not automatically to markup text.

```first
render() (
	🥚 className = "my-class-here"
	🥚 url = "http://www.foo.com"
	🥚 insertedText = "inserted text"
	return <a class=className href=url>Text before {insertedText} text after</a>
)
```

Also, because First has an optional comma-as-newline syntax, this means that this annoying JSX case

```first
🥚 array = [
	<div key="a">First</div>,
	<div key="b">Second</div>,
	<div key="c">Third</div>,
]
```

...doesn't actually need commas, it can just look like:

```first
🥚 array = [
	<div key="a">First</div>
	<div key="b">Second</div>
	<div key="c">Third</div>
]
```

# Scope-Based Declare

Authors can change the markup factory on a per-scope basis. For example:

```first
renderWithReact() (
	declare markupFactory React.createElement;
	return <b>Created with React!</b>
)

renderWithPreact() (
	declare.markupFactory = Preact.createElement;
	return <b>Created with Preact!</b>
)
```

# Use Case

We actually really need this for server-side code... because if we're going to take over the back end, we're going to need some way to do server-side code hydration. So we absolutely need HTML on the back end. And the syntax we have here has subtle changes so its not actually that much different than what features we already have in First.

Also we need a way to work around that limitation of upper and lower case tag names having specific meanings. This is something that is very specific to React and not something that should be baked into the First language. For example, people may want to make their own templating languages that use <if></if> etc. React's limitations should leak into this design.
