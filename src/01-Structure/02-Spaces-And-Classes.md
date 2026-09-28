# Spaces and Classes

Spaces are First's primary mechanism for partitioning programs by responsibility. A space contains the declarations that collectively implement one responsibility and may contain classes, functions, values, types, startup functions, and nested spaces. Spaces may be nested to whatever depth the program's decomposition requires.

Spaces are not merely namespaces. Although they provide qualified names, avoiding naming collisions is not their primary purpose. Source files are likewise not part of this semantic structure: they do not create scopes or divide the program into components. Files are a storage and interchange detail; spaces express the program's architectural boundaries.

Spaces should normally align with responsibility boundaries. Anchors express the responsibilities and prohibited responsibilities of a scope, and most anchors are expected to be owned by spaces. A space that crosses unrelated responsibilities, or divides one responsibility arbitrarily, makes those anchors less precise and less useful.

Spaces and classes share the same declaration syntax. A declaration without a constructor or inheritance clause is a **space**. A declaration with either is a **class**.

This classification applies independently at every depth. Adding a constructor to a declaration reclassifies only that declaration. Its nested declarations keep their identities, but spaces below the new class boundary become groups of that class's instance members until another class introduces a new instance boundary.

```first
App (
	run() (
	)
	
	startup (
		run()
	)
)
```

This `App` is a space because it has no constructor and no inheritance clause. Its members are accessed directly through the space name:

```first
App.run()
```

A declaration becomes a class when it declares a constructor:

```first
User (
	constructor(name is string) (
		this.name = name
	)
	
	name is string
)
```

An inheritance clause also makes a declaration a class:

```first
Admin is User (
)
```

Spaces are not types and cannot be called. Classes introduce nominal types and are called through their constructors.

## Structural Merging

Repeated declarations with the same logical parent, name, and kind merge into one structural declaration. Files do not create scopes, so the merge is based on the program structure rather than where the fragments are stored.

```first
Panel (
	constructor
	title is string = "Inbox"
)

Panel (
	constructor
	readTitle() is string (
		return title
	)
)
```

These fragments declare one `Panel` class. The repeated short `constructor` markers coalesce into the single empty constructor. A class may have at most one implemented constructor form, such as `constructor(...) (...)`. It may not have two constructor bodies or two different constructor signatures.

A constructorless same-name declaration is still a space unless it independently establishes class kind with an inheritance clause. This preserves the paired static side:

```first
Panel (
	constructor
	title is string = "Inbox"
)

Panel (
	readDefault() is string (
		return "Inbox"
	)
)

panel = Panel()
console.log(panel.title)
console.log(Panel.readDefault())
// panel.readDefault(): notice.
```

Merging recursively unions distinct nested members. Duplicate fields, functions, properties of the same accessor kind, implemented constructors, or ghosts produce notices. Distinct getter and setter fragments may form one property. Conflicting parent lists produce a notice; as recovery, later fragments are internally reordered to match the first accepted parent order rather than creating a new order.

Imports belong to the logical merged container. Identical imports may coalesce. Conflicting aliases or package versions in the same container produce notices.

## Empty Spaces

An empty declaration is a non-constructible space:

```first
Configuration (
)
```

There are no implicit constructors for ordinary bare declarations. To declare a class with an empty constructor, write either form below:

```first
User (
	constructor
)

Account (
	constructor() (
	)
)
```

The short `constructor` form declares a constructor with no parameters and an empty body.

Inheritance is the exception: an inheritance clause makes the declaration a class and synthesizes a constructor when none is written.

## Paired Declarations

A lexical scope may contain one space and one class with the same name:

```first
User (
	find(id is string) is User (
	)
)

User (
	constructor(name is string) (
		this.name = name
	)
	
	name is string
)
```

`User.find(...)` accesses the space. `User(...)` constructs the class, and `User` in a type position denotes the class type.

The pair exists when both declarations occur under the same logical parent. Source fragments with the same kind merge with the existing declaration; source fragments of the other kind form or contribute to the pair.

The space and class have independent member surfaces. They share a name but do not create an implicit receiver between their bodies.

Pairing does not change storage ownership. At program scope, the space is the familiar program-owned side of the pair. Under a class, the space belongs to the surrounding instance, while the paired class still declares its own separately constructed instances:

```first
App (
	Panel (
		constructor() (
			Item.count = 0
		)

		Item (
			constructor(name is string) (
				this.name = name
			)
			name is string
		)

		Item (
			count is int
			increment() (
				count += 1
			)
		)
	)
)

panel = App.Panel()
panel.Item.increment()
console.log(panel.Item.count) // 1

item = panel.Item("Subject")
console.log(item.name) // "Subject"
// item.count produces a notice: count belongs to panel, not item.
```

## Nesting and Imports

Both spaces and classes may contain nested spaces, nested classes, and imports. Imports follow the ordinary ordering rules of their logical merged container.

Only spaces may contain startup functions. Only classes may contain constructors.

## Storage Ownership

A space inherits the storage owner of its enclosing declaration. The implicit root and spaces below it own program-lifetime state. A space below a class groups members belonging to that class instance. Further nested spaces preserve the same owner until a nested class introduces a new instance boundary.

```first
Counter (
	constructor
	In (
		count is var int = 0
	)
)

a = Counter()
b = Counter()
a.bump()
console.log(b.read()) // 0
```

The path is part of each grouped member's address. `a.In.count` does not create an alias named `a.count`. A space adds no allocation, identity, receiver, or runtime value of its own, so a space cannot be assigned, passed, returned, compared, or replaced. Compiler-side code may inspect its declarations through the compiler's reflection facilities.

Nested spaces and nested classes below a class are protected structural partitions. Their paths are available to the owning class, structurally nested class bodies with an explicit receiver, and nominal descendants. They are not public instance paths:

```first
Counter (
	constructor
	In (
		count is var int = 0
	)
	bump() (
		In.count += 1
	)
	read() is int (
		return In.count
	)
)

counter = Counter()
counter.bump()
console.log(counter.read())
// counter.In.count: notice.
```

Direct ordinary members of a returned nested instance remain public. Protection applies to the route into the containing class's implementation structure, not to every member of every value whose type was declared beneath that class.

Adding or removing a constructor recomputes storage ownership below that declaration up to the next nested class. Typist preserves declaration identity and folding state and reports accesses that became invalid; it does not synthesize an instance or rewrite those accesses automatically.

## Nested Classes

A nested class introduces a nominal type and a new instance boundary. It is not automatically instantiated and does not capture an instance of its enclosing class. Runtime construction requires an instance of the immediately owning class:

```first
Panel (
	constructor
	Item (
		constructor
		name is string = "item"
	)
)

panel = Panel()
item = panel.Item()
// Panel.Item(): notice outside Panel; construction requires protected Panel authority.
```

The declaration-qualified path remains valid in type positions and parent lists because naming a type performs no runtime access:

```first
read(item is Panel.Item) is string (
	return item.name
)
```

Two `Panel` instances expose the same nested nominal type. This receiver requirement does not create object-dependent types or give `Item` an implicit reference to its `Panel`. A nested class that needs outer instance state must receive or store an explicit reference. Structurally nested class bodies may use that explicit receiver to access the outer class's protected partitions:

```first
Panel (
	constructor
	In (
		count is int = 7
	)
	Item (
		constructor
		read(outer is Panel) is int (
			return outer.In.count
		)
	)
)
```

An instance-owned space cannot be crossed through a declaration path to bypass this rule. If `Item` is nested under `Panel.In`, then `Panel.In.Item()` is invalid because `Panel.In` has no program-owned member surface. Construction must begin from a `Panel` instance.

Names are qualified with dots:

```first
App.Backend.User
```

Named spaces and classes cannot use dotted declaration names. Nesting must be written explicitly.

```first
App (
	Backend (
	)
)
```

## Member Name Lookup

Inside a space or class, a member may be named without a qualifier. Name lookup checks local bindings and parameters first, then members of the current space or class, then names in enclosing scopes. An inherited class member is a member of the current class for this purpose. Lookup proceeds upward only; it does not search sibling or descendant spaces. A nearer binding shadows an outer one even when it is unsuitable for the attempted operation.

Within a class instance boundary, including inside any of its spaces, `this` is the owning class instance. Spaces add lexical scopes without adding receivers. Entering a nested class changes `this` to the nested instance. Use `this` when a local or parameter hides an instance member; grouped members retain their full path, such as `this.In.count`.

Nesting a class inside another class does not make either instance available to the other. Code cannot reach an enclosing class's instance members without an explicitly stored or passed reference, and an outer instance cannot reach a nested class's instance members without constructing or receiving a nested instance. Program-owned declarations remain available through ordinary lexical lookup and qualification.

## Names

Space and class names begin with a capital letter. Definitions such as selection types, aliases, and proofs follow the same naming family. Runtime members such as functions and fields begin with a lowercase letter or another permitted runtime-name prefix.

Spaces and classes cannot be anonymous. The only unnamed structural space is the implicit root space that contains the project.

## Editor Rendering

Typist renders a space with a small folder-style badge and a class with the ordinary class rendering. When a space and class share a name, Typist places them adjacently.

The badges are editor projections and are not persisted as source tokens.

## Qualified Space Locals

Every binding declared directly in a space has a qualified address. Function and block locals remain lexical locals; they do not become members of the containing space. Qualification is independent of visibility or access policy.

```first
Settings (
	small = 16

	Width is one of (
		this.small
		32
	)
)
```

In program-owned space code, `this` qualifies the nearest containing space but is not itself a runtime value. In class instance code, including instance-owned spaces, it denotes the current instance. A selection entry list does not introduce its own receiver. `this.name` does not search outward through program spaces; outer space members require explicit named qualifiers such as `Settings.small`. The implicit root is a space too.

Class instance members require an instance; the class name cannot qualify a space-local value that no longer exists after the declaration becomes a class. Paired space/class declarations retain their independent member surfaces.
