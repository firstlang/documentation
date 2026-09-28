# Local Bindings

## Space Qualification

Space-local bindings are addressable through their space, without an additional declaration or export syntax. Function-locals and block-locals remain local to their lexical scope. Qualification and visibility are orthogonal.

```first
Settings (
	small = 16

	Width is one of (
		Settings.small
		32
	)
)
```

A selection may capture a direct immutable literal value, but not a computed initializer or runtime-dependent value. A direct reference does not copy its binding name into an unnamed selection. See [[11-Selection-Types]].

## Earlier Local Design Notes

Identifiers are always immutable. First also doesn't support variable shadowing. Therefore, we can support emoji-based variable declarations, like so:

```first
🥚 x = 1
🥚 y = 2
```

Another declaration in the same scope causes a notice to appear

```first
🥚 a = 1
a = 12 // Notice: you can't make a = 12 because it was already set to 1.
```

Mutable variables need to be annotated as "var" explicitly. This is true both in argument lists, as well as in control flow.

```first
🐝 a is var = 1
a = 12 // No notices
```

If you try to re-assign an identifier in a scope, it just makes a new one:

```first
🥚 x = 1

1 to 10 each i (
	// It makes a new identifier here
	// It probably also fades out the other x variable
	// So that people know that it's not the same thing,
	// when the focus is inside this scope.
	🥚 x = 2;
)

1 to 10 each i (
	// This would be unrunnable code, because "x already has a value". 
	// The environment would ask you if you want to make the value variable.
	x += 2 
)
```

You have to annotate something with "variable" in order to actually make it mutable. Also, it doesn't just auto-flip to variable. It asks you if you want to make it variable. Such as:

```first
🐝 z is var = 3
```

The `is var` annotation looks long and ugly because it should. It also gives you help about why its easier if you keep things not variable. The `: variable` annotation is also necessary in parameter lists:

```first
foo(x is var int, y is var int)
{
	x += 1
	y += 1
	return x + y
}
```

Also, this has immediate visual differentiation, because they're going to see something that looks much different, and it re-enforces the "kids can use this, but you can write avionics controls systems in this".

You can't variable shadow in First like you can in Rust. This wouldn't work in TypeScript anyway. Also, I don't know how you'd get that understood or how that would work in a structured editor. Different emoji colors? It gets ugly fast. Also it screws up the structured editor UI and workflow.

Perhaps just use `var` instead? We already have this for mutable arrays

```first
foo(x is var int, y is var int)
{
	x += 1
	y += 1
	return x + y
}
```

> ⚠️ The problem with this design is that it collapses mutable references and mutability in arrays. Thats' weird.

# Constructor Parameters And Fields

Constructor parameters are ordinary local bindings inside the constructor body. They are not promoted by visibility markers. Store a parameter by assigning it to an explicit field.

```first
Baz (
	constructor(a is int, b is int) (
		this.a = a
		In.b = b
	)
	
	a is int
	In (
		b is int
	)
)

baz = Baz(1, 2)
console.log(baz.a) // Logs 1.
// baz.In.b: notice.
```

# Shadowing

This design is a bit odd, but is much more ergonomic. We allow scope-based shadowing of immutable hatches, and no shadowing on mutable hatches.

```first
foo() (
	🥚 x = 10
	🐝 y is var = 20;
	
	if (Math.random() > 0.5) (
		
		// This gets you a new variable, because
		// you can shadow when you're immutable.
		🥚 x = 20
		
		// This still uses the same non-shadowed variable.
		// The editor will need to draw this out so that authors
		// can follow that it's the same mutable variable as before.
		// I'm leaning towards this design.
		y = 10
	)
)
```

The downside, of course, is different shadowing rules for mutable and immutable variables. However, editor help likely mitigates the downside significantly, as well as the 🐝 emoji which suggests that the mutable variable is now "flying around everywhere, and perhaps a bit dangerous".

# No Same-Scope Shadowing

Unlike Rust, shadowing a variable within the same scope is prohibited. This leads to developer confusion (especially in rust where the shadowing variable can be declared with completely different types). Also, much of the need for this is addressed in self-referencing assignments (discussed further down).

```first
foo() (
	🥚 x = 10
	console.log(x)
	x = 20 // ❌ Compiler notice
)

```

# No Mutable Closure Variable Assignments

Closures would only have read-only access to higher-scoped identifiers. This is better because lowering this kind of thing to Rust sucks anyway. Also its a huge source of bugs. The added verbosity doesn't matter that much either because AI will get good at generating it.

```first

foo()
{
	🥚 x = 10
	🐝 y is var = 20;
	
	fn = () =>
	{
		// This makes a new value, it doesn't access the parent one.
		// We might need a frozen egg instead of a normal one
		// for the image, provided we're willing to break from the
		// unicode standard emoji set.
		🥚 y = 30
		console.log(y); // 30
	}
	
	console.log(y); // 20
	fn();
}

```


# Valid Identifier Regex

Hatches follow the same identifier naming restrictions as found in TypeScript.


# Hatch Self-Referencing

## Rationale

Self-referencing assignments are an **affordance for immutability**, making first-time variable creation more powerful and beginner-friendly. They eliminate a common friction point: constructing an immutable value often requires either:

1. Creating a nested scope with a temporary mutable variable.
2. Breaking immutability by declaring the variable `var`.

With self-referencing assignments:

- The compiler automatically detects the entire series of sub-expressions that precede the sub-expression that contains the self-reference, and applies a compound assignment.
- Developers can write expressive, concise initializers without sacrificing immutability.
- Code remains readable and predictable, with clear compiler notices when the expression is nonsensical.

This feature is especially useful in educational contexts, top-down programming patterns, and AI-assisted workflows in Tokens, where immutability is the default. It reduces boilerplate, prevents unnecessary mutability, and encourages a cleaner mental model for variable initialization.

Self-referencing assignments provide a shorthand for initializing a variable with a complex expression that references itself. They allow the variable to remain immutable while avoiding boilerplate temporary variables. First-time declarations are visually indicated by the 🥚 icon in Tokens.

## Self-Referencing Rules

1. **Scope:** Applies **only to first-time declarations** (🥚).
2. **Trigger:** Only **two-operand operators** are eligible:

```
+  -  *  /  %  **  |  &  ^  <<  >>  >>>  or  and
```

3. **Cumulative base assignment:**

   - The compiler identifies the **last sub-expression containing the variable itself**.
   - Everything **before that sub-expression** in the top-level expression becomes the **base assignment**.

4. **Compound assignment:**

   - The identified sub-expression containing the variable becomes a **compound assignment** applied to the base.

5. **Unsupported patterns / notices:**
   
   - Operators outside the supported list generate a **compiler notice**.
   - Expressions with no variable reference in the last sub-expression generate **notices**.
   - Closures nested within assignment expressions **do not trigger** self-referencing assignments.

## Self-Referencing Examples

```first
// 🥚 indicates first-time declaration
// 🥚 indicates susequent declaration
🥚 x = a() + b() + c() + d() + 🥚 x > 10 ? 0 : 1

// Expands to:
x_1 = a() + b() + c() + d()
x = x_1 > 10 ? 0 : 1
```

```first
🥚 y = a() + (🥚 y + 🥚 y)

// Expands to:
y = a()
y += y + y
```

```first
🥚 z = (z + 1) * 2

// Nonsensical: expands to default with compiler notice
z = (0 + 1) * 2
```

```first
// Null coalesce:
🥚 x = a() + b()🛡️.c else 🥚 x

// Expands to:
x_1 = a()
x += b()?.c else x_1
```

```first
// Nullable operator
compute(): int | null { ... }

🥚 value = compute() + (🥚 value else 0)

// Expands to:
value_1 = compute()
value += value_1 else 0
```

## Rationale

Self-referencing assignments are designed as an **immutability affordance**, making first-time variable creation more expressive and beginner-friendly. They allow:

- Automatic splitting of cumulative expressions into a base assignment and a carried compound assignment.
- Avoiding unnecessary mutable temporaries or breaking immutability.
- Cleaner, more readable code, especially for complex initializations.
- Clear compiler notices when the expression is nonsensical or unsupported.

This feature is particularly useful in top-down programming, educational contexts, and AI-assisted workflows in Tokens, where immutability is the default. It reduces boilerplate, clarifies intent, and aligns variable creation with the natural flow of cumulative expressions.
