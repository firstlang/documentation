# Anchors

Anchors are compiler-aware natural-language sentences written in First source. They describe what nearby code is supposed to mean, do, preserve, or still decide. They are program context, not comments: the compiler parses them, assigns them to their enclosing declaration, classifies them, and exposes them to tools.

Anchors are primarily authored by language models and maintained through the editor. They should therefore use a deliberately small syntax that is easy to generate, easy to check, and hard to confuse with ordinary First code.

An anchor is not a proof, contract, invariant, test, or formal postcondition. Deterministic requirements that the compiler can check directly belong in those mechanisms. Anchors exist for intent, responsibility, generation guidance, partial implementation points, and visible uncertainty.

## Syntax

An anchor begins with `-` at a structural boundary:

```first
PaymentRecovery (
	- Keeps declined payments understandable to the customer.

	retryPayment() (
		- Load the current payment attempt.
	)
)
```

The dash is the only serialized anchor marker. The old `#` fragment syntax, addressed fragments, anchor headings, nested anchor trees, and inherited fragment spines are not part of the language.

Each serialized anchor is one physical line. It ends at the line break. Long or compound intent should be split into multiple anchors or rewritten into one concise sentence.

Anchors may appear wherever the enclosing construct permits structural items or executable body items. Placement determines ownership. A semantic reference inside an anchor does not transfer or share ownership. Some declarations that do not otherwise need executable contents may still use an anchor-only body so the anchors have a hard owner.

Comments are separate. A `//` comment may visually group nearby anchors, but it does not own, classify, scope, inherit into, or otherwise affect them.

```first
// Customer-visible recovery
- Keeps recovery language consistent with checkout.
- Should failed gift-card payments use the same retry path?
```

## Text Characters

Raw anchor prose is intentionally restrictive.

Anchor text may contain:

- Unicode scalar values whose General Category is Letter (`L*`), Mark (`M*`), or Number (`N*`), as defined by the Unicode Character Database.
- Ordinary space characters between words.
- The punctuation characters `.`, `,`, `'`, and `-`.
- A question mark only as the final non-whitespace character of a pending question anchor.
- Brace-delimited semantic islands, as described below.

Tabs are allowed only as indentation before the anchor marker. Raw tabs are not allowed inside anchor text.

The compiler must reject unassigned code points, private-use characters, surrogate code points, control characters, format characters, default-ignorable characters, emoji pictographs, dingbats, symbol characters, and operator punctuation unless a later feature explicitly admits them through a dedicated subgrammar.

There is no general escape syntax for anchor prose. Reserved characters, including `:` and `;`, should be expressed in words or placed inside a semantic island when the island grammar permits them.

## Pending Questions

If the final non-whitespace character of an anchor is `?`, the compiler classifies it as a pending question anchor. Raw `?` is not permitted anywhere else in anchor prose.

```first
- Should failed gift-card payments use the same retry path?
```

Pending question anchors are compiler-recognized unresolved decisions. The language does not constrain what kind of question may be asked beyond the ordinary anchor character rules.

Resolving a pending question is an editor or workflow action, not a language lifecycle. Source should eventually contain the accepted anchor text, but the compiler does not prescribe how history, provenance, or conversation records are stored.

## Declarative, Imperative, And Uncertain Anchors

Accepted non-question anchors are classified as declarative, imperative, or uncertain.

A **declarative anchor** states a fact, responsibility, invariant-like intent, constraint, or guardrail about its enclosing declaration.

```first
- Payment failures remain explainable without exposing processor internals.
```

An **imperative anchor** gives an instruction at an executable source position. It may be replaced by generated implementation at that position.

```first
retryPayment() (
	- Load the current payment attempt.
	- Return the normalized retry outcome.
)
```

An **uncertain anchor** is an accepted anchor whose wording cannot be reliably classified as declarative or imperative. The compiler reports a notice for uncertain anchors. They remain part of the program because uncertainty should stay visible instead of being silently resolved by tooling.

The compiler may use light natural-language classification to distinguish these forms. The classification is compiler-visible, but it is not expressed with additional source syntax.

Imperative anchors may appear only inside executable bodies: functions, constructors, and startup functions. Declarative anchors may appear in structural or executable contexts according to the ordinary placement rules for anchors. Field anchor bodies and other anchor-only structural bodies do not admit imperative anchors, because they do not execute.

## Semantic Islands

Braces introduce a semantic island inside an anchor. A semantic island is checked by the Partial Verifier rather than by the raw anchor-prose character rules.

```first
- Keeps offline work usable when {Network} is unavailable.
- Makes initialization failures from {calculateThings(...)} actionable.
- Preserve the behavior described in {chat:2f4a9d87}.
- Compare the current layout against {image:reference%20layout.png}.
- Follow the accessibility guidance at {https://example.com/a11y}.
```

The final `?` rule for pending questions applies to the anchor as a whole, outside islands. A question mark inside an island does not classify the anchor as pending.

There are two island forms:

- A **partial-code island** contains ordinary First syntax, a name reference, member reference, type reference, call shape, or another Partial Verification fragment.
- A **resource island** contains an external resource reference.

Resource islands use this form:

```text
{scheme:payload}
```

`scheme` must begin with an ASCII letter and may continue with ASCII letters, digits, `+`, `.`, or `-`. The language does not require any specific scheme such as `chat`, `image`, `file`, `http`, or `https` to be supported by every implementation.

The `payload` is an opaque UTF-8 payload interpreted by the implementation that understands the scheme. Characters with special meaning to the island delimiter or to plain source text must be percent-encoded using URL-style percent encoding. In practice, this follows `encodeURIComponent`-style escaping, including literal `{` and `}`, encoded as `%7B` and `%7D` so they cannot terminate or nest the island accidentally.

Unknown schemes remain valid resource islands. Editors may render supported schemes inline, as links, as previews, or as context attachments. The compiler must not require fetching or executing an external resource to parse the program.

## Partitions

Partition references are separate from anchor identity. Anchors no longer expose stable numeric identities or `#123` addresses in source.

An anchor can still be the source position for a partition. The enclosing context determines what kind of partition is allowed:

- Imperative anchors in executable bodies may attach body partitions. The partition contributes implementation at the anchor's source position.
- Declarative anchors in structural contexts may attach structural or member partitions. The partition contributes lower-stratum declarations associated with the enclosing declaration.
- Uncertain anchors may attach partitions only when their enclosing context determines a valid partition kind. For example, an uncertain anchor inside a function may attach a body partition, but an uncertain anchor directly under a space cannot attach an executable body partition.

These rules keep partition implementation tied to the anchor that describes its responsibility without making anchor identity syntax part of the language.

## Ownership

Every anchor has exactly one structural owner: the directly enclosing declaration or executable body that contains it.

The implicit root space owns anchors written at the top level. Source files do not independently own anchors.

The following constructs may own anchors:

- Spaces, including the implicit root space
- Functions
- Startup functions
- Classes
- Fields, when written in expanded form
- Extension classes
- Primitive classes
- Constructors
- Properties
- Ghosts
- Selection types
- One-of declarations
- One-value-of declarations
- One-case-of declarations
- Many-of declarations

Imports, parameters, locals, and statements do not own anchors.

Fields use an optional anchor body after the complete field declaration when they contain anchors. The body attaches to the field as a declaration, not to the initializer expression. It creates no runtime scope, receiver, or namespace, and it permits anchors plus ordinary `//` comments only.

```first
value is int = 12 (
	// Ranking
	- Keeps ranking changes explainable when this confidence score changes.
)
```

Code, declarations, or compiler directives written in a field anchor body produce notices and have no execution or declaration effect. The field, initializer, and valid anchors remain usable.

Although locals may share some field syntax, the editor does not expose anchor ownership for locals.

## Tooling And Evaluation

Anchors are source-level intent. Evaluation states such as supported, contradicted, inconclusive, stale, unevaluated, approved, or rejected are tooling metadata over anchors, not anchor syntax and not language-level states.

Generated code must not silently delete, weaken, or rewrite an anchor to make its output appear compliant. If an implementation cannot satisfy an anchor, the discrepancy should remain visible to tooling and review.

Tooling may retain history, provenance, authorship, conversation records, evaluation evidence, approval state, priority, or presentation state. None of that metadata is serialized as anchor grammar.

## Relationship To Other Features

- Comments use `//` and are non-semantic prose.
- Invariants established with `declare` alter compiler behavior deterministically.
- Contracts, including `ensure` postconditions, and tests are deterministic language mechanisms.
- Partial Verification checks semantic islands used by anchors and examples.
- Partitions may attach implementation or lower-stratum declarations to eligible anchors.
