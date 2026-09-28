
_Design document. Captures the staging model, the compile/runtime bridge, the reflection tier, and the provenance model that makes generated code reviewable in Typist._

## 1. Motivation

First is meant to compile to native code while staying fully sound and always-compiling. Separately, Typist is meant to let AI generate large bodies of code that a human can still understand and direct rather than accept as a black box.

Those two goals meet at a single feature: **compile-time code that can construct and mutate the program itself**, with the compiler recording enough about that construction that a reviewer can trace any piece of generated code back to exactly what produced it — not by static analysis after the fact, but because the compiler watched it happen.

This is the same idea as the "generation chain" model (codegen as a sequence of reviewable stages, each a projection of the last) applied specifically to First's own compile/runtime boundary.

## 2. The staging model

Staging isn't a ladder of trust tiers — it's a small set of rules about identifiers and where they're declared. Any statement can be pushed onto the compile side by leading it with `!` followed by a space. This doesn't change its semantics; it asserts that everything it depends on is statically known, and evaluates it early. The marking applies to the whole statement: because of how the parser scopes what follows, `!` placed before a function pulls that function's entire body — including its opening and closing parentheses — along with it into the compile-time statement.

### 2.1 Non-staged constructs

Some constructs never have a runtime existence at all: `one of`, type aliases, and similar declarative forms. These require no `!` marking, because their meaning is definitionally independent of anything only known at runtime. There is no "compile-time version" of `one of` — there is only one version, and the staging rules below simply don't engage for them.

Selection entries are restricted to literal values, direct references to such values, and type alternatives. General constant evaluation is not enabled by declaring a selection: arithmetic and calls remain excluded even when referentially transparent. An unmarked space-local initialized directly with a permitted literal may be referenced by a selection; this is declarative value resolution, not permission for arbitrary staged code to read runtime bindings. Resolve direct-reference dependencies after generated spaces seal.

### 2.2 One namespace, one declaration site

Identifiers on the compile side and identifiers on the runtime side share a single namespace. A name may be declared on only one side; declaring it again on the other side in the same scope is a redeclaration error, not a shadow. Two consequences follow:

- **No same-name collisions to reason about.** `foo` and `! foo` can't coexist as two different bindings in the same scope — you're forced to pick a different name for one of them, which avoids the confusing case of two things that read almost identically meaning different things.
- **A name's stage is fixed at its declaration, forever.** Whether a binding lives at compile time or runtime is decided once, by whether its declaration is `!`-led or not. It is never re-marked at use sites, on either side. `! day = Day.tuesday` declares `day` as compile-time; every later reference — from compile-time code or runtime code — is just `day`.

This is also why no special "read a compile-time value" syntax is needed. Since a name resolves to exactly one binding regardless of which side declared it, there's nothing to disambiguate at the use site. (It also sidesteps an actual syntax collision: `!` already means boolean negation, so a hypothetical `!day` marker at a use site would be genuinely ambiguous with "negate the boolean `day`.")

```
foo() (
	! day = Day.tuesday
	
	! if (day > Day.monday) (
		console.log("Looks like day was after monday at compile time...")
	)
)

Day is one of (
	monday
	tuesday
	wednesday
)
```

### 2.3 Reading across the boundary is asymmetric

- Runtime code reading a `!`-declared name: always fine. By the time runtime code runs, that name isn't a variable anymore — it's been resolved to a value and substituted at every site that reads it, before the compile-time space seals. Reading a compile-time result from runtime is this substitution, not a function call or a special access mode.
- Compile-time code reading an unmarked (runtime-declared) name: never fine. The value doesn't exist yet at compile time; there's nothing to read.

### 2.4 Calling into unmarked runtime code requires referential transparency

Compile-time code may call an ordinary, unmarked runtime function only if that function is **referentially transparent**: no reads of mutable ambient state, no I/O, no clock, no randomness — output determined entirely by arguments. This is checked transitively, the same way effect systems check that a function's effects are a subset of what's granted.

This isn't a separate category of function — any ordinary function is just a function. The restriction lives at the call site, on the compile-time side, as the price of proving it's safe to invoke early. Purity is what makes a function usable from both sides without being written twice; it deliberately doesn't extend to anything that touches disk or network — that kind of effectful generation belongs to a different stage entirely (see the companion generation-chain document).

### 2.5 `!`-declared functions have no runtime existence

This isn't a withheld permission — there's simply nothing left to call. By the time codegen finishes, a `!`-declared function has already been executed and is gone; there's no artifact for runtime code to link against. Asking whether runtime code can call it is like asking whether generated code can call the macro that generated it.

### 2.6 Compiler-API effects are ambient, not a gated tier

Inside compile-time code, calls like `Space.append` and constructor calls that produce a code-DOM representation (Section 4) are simply things compile-time code can do — the same way runtime code can do I/O without invoking a special keyword. There's no separate, more-trusted syntactic form for "reflection code" versus ordinary compile-time code; what makes arbitrary construction and mutation safe to expose is uniform across all compile-time code, and comes from interpreter tracing (Section 5), not from gating a subset of it behind different syntax.

Compile-time code follows the same structural access rules as runtime code. It may inspect declarations through the compiler API when that API represents source structure, but it does not gain special permission to read or execute through protected runtime paths. Generated source receives the authority of the logical declaration where it is emitted, just like authored source.

## 3. Types as compile-time values

Reflection code needs to pass types around as ordinary values (e.g. `Field("description", string)`). First's generics already use a types-as-values syntax, so this falls out of the existing design rather than requiring a new mechanism — a type reference is just a value like any other argument.

## 4. Spaces as mutable collections

Spaces become mutable, appendable collections when accessed from compile-time code. A space starts empty or with its ordinary declared contents, and compile-time code can push newly constructed definitions into it.

```
RuntimeObject (
	constructor
	
	name is string
	value is string
)

App (
)

// ! startup gives you a compile-time entry point
! startup (
	ro = 🏗️ RuntimeObject()
	ro.fields.push(Field("description", string))
	App.append(ro)
)
```

### Stage-polymorphic construction

First has no `new` keyword — constructing something is just calling it, e.g. `RuntimeObject()`. What that call produces depends on which side calls it:

- At runtime, `RuntimeObject()` allocates an ordinary instance.
- At compile time, a constructor call produces a **builder / code-DOM representation** of the class — something you can mutate (add fields, rename, etc.) before it's appended anywhere. Typist renders this with a leading 🏗️ before the call (`🏗️ RuntimeObject()`), so the distinction is visible directly in the editor rather than only inferable from context.

The same syntax produces a different _shape_ of result depending on stage — a bigger asymmetry than ordinary staging, where `!`-evaluation usually just means "the same semantics, resolved early." In a plain text editor this ambiguity would be a real hazard (code reading `ro.fields.push(...)` could be misread as mutating an object's data rather than a class's shape). Typist's 🏗️ marker resolves this at the point of construction, so the ambiguity never reaches the reader in the first place — it's a rendering concern, not a type-system concern.

### Sealing

A space stops accepting appends at the end of its `! startup` block, once there are no outstanding callbacks left to resolve. A generation timeout guards against run-on generation (e.g. a callback chain that never terminates) so sealing is always reached. After sealing, a space's contents are fixed and ordinary static typing proceeds against them as if they'd been declared directly.

## 5. Provenance via interpretation, not static analysis

Because `! startup` blocks and other compile-time code are **run by an interpreter** rather than compiled opaquely, the compiler observes every step of construction as it happens: every 🏗️-marked constructor call, every mutation (`ro.fields.push(...)`), every `append`. This gives a complete, literal chain of history from the compile-time code that initiated a value to wherever it landed in the final program — recorded as the run happens, not reconstructed afterward.

This is a materially different (and easier) problem than the one the generation-chain document originally posed. That document assumed provenance would have to be **static** — given a piece of generated output, work backward through the generator's source to find the rule that must have emitted it. That's tractable for template-shaped generation but breaks down for arbitrary reflection, where there's no clean rule-to-output mapping to point at.

Interpreted execution sidesteps the inverse problem entirely: the compiler already knows what produced a value, because it watched the value get produced. This is what makes unrestricted compile-time construction compatible with Typist's fault-localization goals — the power and the traceability are no longer in tension, because traceability doesn't depend on the generation being structured. It depends on the compiler being the one executing it, step by step, with a record kept.

This also means the trace shown to a reviewer is a property of one run, not a static property of the generator — if a `! startup` block branches on something that can vary between runs, the chain-of-history view is a debugging/replay artifact tied to that run, not something Typist can present without having executed the generator at least once.

## 6. Why two different restrictions solve two different problems

It's worth keeping the referential-transparency restriction (2.4) and interpreter tracing (Section 5) conceptually separate, because they answer different questions:

- **Referential transparency** buys reproducibility. A bridge function that's pure gives the same output on every build; nothing about tracing execution helps if the function itself is nondeterministic.
- **Interpreter tracing** buys provenance. It tells you exactly what happened on a given run, but says nothing about whether the next run will do the same thing.

Compile-time code isn't required to be referentially transparent itself — it's allowed to do far more than calling into pure runtime functions permits. What makes that safe to expose in the editor isn't a purity restriction, it's that every run is fully observed.

## 8. Open questions

1. **Hole identity for template-generating functions.** A related but distinct pattern (a class definition with bare `!` holes standing in for constructor parameters, producing a portable, fillable class value) needs named holes if the same hole is meant to be reused at multiple positions (e.g. a field name reused in a method signature). A bare, unnamed `!` only works cleanly if each occurrence is guaranteed to be a distinct parameter.
2. **Typed holes vs. raw AST.** Whether the "class with holes" produced by such a function should be a first-class typed value (a declared hole signature, checked before it's filled) rather than untyped syntax — the former stays consistent with First's "no escape hatches" goal; the latter risks the stringly-typed pitfalls of text-based macro systems.
3. **Nested compile-time calls.** If compile-time code's output can itself contain further compile-time constructor or reflection calls, an expansion order needs to be chosen (single pass, fixpoint, explicit stage numbering) or ordering bugs of the kind proc-macro systems run into become possible.
4. **Multiple entry points into one space.** If more than one `! startup` (or other compile-time entry point) can append into the same space, sealing has to wait for all contributors, and an ordering/determinism story is needed across them — the same class of problem as nested expansion, one level up.
5. **Compiler-side imports**. Should authors be able to `! import` code on the compiler side like they can on the runtime side?
