# Classes and Inheritance

A declaration with a constructor or inheritance clause is a **class**. A class defines a nominal type, creates values, and provides a qualified scope for instance members.

```first
User (
	constructor(name is string) (
		this.name = name
	)
	
name is string
	
displayName() is string (
	return name
)
)
```

Calling the class invokes its constructor. First has no `new` keyword.

```first
user = User("Ada")
```

Classes are reference types by default. Primitive classes define value types and belong to a separate type universe.

## Fields

Stored fields are class or space members that begin with a runtime member name. A field may omit its type annotation; a bare field is equivalent to a field annotated as `unknown`.

```first
Session (
	authentication
	identifier is int
)
```

When a field has an initializer but no explicit type, its type is inferred from the initializer.

```first
Status (
	label = "ready"
)
```

Field initializers are intentionally restricted to a literal value or a single named value. Function calls, operators, and formula-like expressions belong in constructors or executable bodies instead of field declarations.

```first
Counter (
	count is int = 0
	state = InitialState
)
```

Fields may use an anchor-only body after the complete declaration. That body may contain anchors and `//` comments, but it is not executable.

```first
Counter (
	count is int = 0 (
		// Persistence
		- Starts at zero for newly created counters.
	)
)
```

## Inheritance

Inheritance uses `is` and makes the declaration a class even when it has no explicit constructor.

```first
Animal (
	constructor
	
	eat() (
		console.log("animal eats")
	)
)

Bunny is Animal (
)
```

`Bunny is Animal` means `Bunny` is a nominal subtype of `Animal`.

Inherited members behave like structural embedding: members from the parent classes are copied into the child. The child retains the nominal history of the inheritance graph, so a value can be viewed through any inherited nominal type.

There is no virtual or dynamic dispatch. A value accessed through `Animal` uses the `Animal` member. A value accessed through `Bunny` uses the `Bunny` member.

```first
animal is Animal = Bunny()

animal.eat()
(animal is Bunny).eat()
```

The expression `(animal is Bunny)` is an implicit short-circuiting cast. If the value is not a `Bunny`, that path does not continue.

## Overrides

When a child declares a member with the same name as an inherited member, it overrides that member for the child type.

```first
Animal (
	constructor
	
	eat() (
		console.log("animal eats")
	)
)

Bunny is Animal (
	eat() (
		super.eat()
		console.log("bunny eats")
	)
)
```

Overrides are inferred by matching an inherited member. Typist renders an override marker before the declaration.

Override signatures may widen or narrow parameter and return types because override dispatch is not polymorphic. The member selected by a call is determined by the nominal type through which the value is accessed.

## Multiple Inheritance

A class may inherit from multiple classes.

```first
Bunny is Animal, Pet (
)
```

Parent order matters. Later inherited members are assigned into the child after earlier inherited members. Duplicate inherited names are permitted because each inherited member retains its nominal origin and may be reached through the corresponding parent view.

Spaces inherited as members merge recursively by name. The merge preserves every inherited member's nominal origin, storage, and lexical bindings. An inherited function continues to resolve unqualified names in the environment where it was declared; grouping does not introduce virtual rebinding. At a colliding non-space leaf, ordinary nominal-view dispatch and later-parent precedence apply.

```first
Base (
	constructor
	In (
		a is int = 1
		label() is string (
			return "base"
		)
	)
)

Child is Base (
	In (
		b is int = 2
		label() is string (
			return super.In.label() + " child"
		)
	)
)

child = Child()
console.log(child.In.a) // 1
console.log(child.In.b) // 2
console.log(child.In.label()) // "base child"
```

Inside an instance-owned space, `super` still denotes the owning class's inherited view. The complete group path is required; it does not gain a group-relative meaning.

Inheritance parents are classes. A non-constructible space cannot be used as a parent merely to share its declarations.

Protected nested partitions follow the declaring class, not the final merged child view. A child body may access protected partitions owned by any of its ancestors through any compatible receiver. An inherited parent body keeps the authority and bindings it had where it was declared; it does not gain access to child-only partitions or unrelated-parent partitions merely because a child merges those groups.

```first
Left (
	constructor
	In (
		left is int = 1
	)
	readLeft(other is Left) is int (
		return other.In.left
	)
)

Right (
	constructor
	In (
		right is int = 2
	)
)

Combined is Left, Right (
	readBoth(other is Combined) is int (
		return other.In.left + other.In.right
	)
)
```

`Combined.readBoth` can see both inherited protected partitions. `Left.readLeft` keeps its original `Left` authority and does not acquire a binding for `In.right`.

First does not have abstract classes, abstract constructors, or abstract members. A declaration without a constructor remains a space. Bodyless functions and members are ordinary incomplete declarations with fallback behavior, not descendant implementation requirements.

## Nested Class Refinement

An inherited nested class keeps its declaration identity. A local same-name nested class refines it through inheritance rather than replacing it with an unrelated type. The local declaration must first be a class under the ordinary classification rule: a constructorless declaration with no inheritance clause remains a paired space instead.

When several outer parents contribute a same-name nested class, the local refinement inherits all of them in outer-parent order. Explicitly named parents follow those implicit parents in written order. Repeating the same parent explicitly does not add it twice. Ordinary later-parent precedence then applies.

```first
Left (
	constructor
	Item (
		constructor
		left is int = 1
	)
)

Right (
	constructor
	Item (
		constructor
		right is int = 2
	)
)

Extra (
	constructor
	label() is string (
		return "extra"
	)
)

Combined is Left, Right (
	Item is Extra (
		constructor
	)
)

combined = Combined()
item = combined.Item()
console.log(item.left) // 1
console.log(item.right) // 2
console.log(item.label()) // "extra"
```

Declaration-qualified nested class paths such as `Left.Item` are valid in type positions. They name a type and do not construct it. Runtime construction still requires an instance of the class that owns the nested declaration.

A protected nested class may be used as an inheritance parent only where authority over its containing class's protected route is available. Code outside that structural or nominal class family may name the type in annotations and use returned values, but it may not derive a public constructor from that nested class.

```first
Panel (
	constructor
	Item (
		constructor
		name is string = "subject"
	)
	createItem() is Panel.Item (
		return Item()
	)
)

Replica is Panel.Item (
)
// Notice: root has no authority to inherit from Panel.Item.

CustomPanel is Panel (
	Item (
		constructor
	)
)
```

A nested class does not implicitly receive an enclosing class's generic constructor arguments. It declares and receives its own generic arguments under the ordinary generic rules. Nesting alone supplies neither a hidden outer construction nor generic propagation.

## Static Inheritance Cycles

First rejects inheritance that would make a class's nested static surface contain itself recursively. After resolving explicit parents and implicit same-name nested parents, the compiler considers class-parent edges together with nested-owner edges. An inheritance edge is invalid when its parent is the class itself or one of its static owners, or when adding it creates any cycle in that combined graph.

```first
Outer (
	constructor
	Bad is Outer (
		constructor
	)
)
// Bad's parent edge is rejected because Outer owns Bad.
```

The same check catches indirect cycles across several classes and nested declarations. The compiler reports a notice and drops the invalid inheritance edge before expanding inherited nested surfaces. Sibling classes may inherit from one another when the combined graph remains acyclic. Spaces, fields, constructors, startup blocks, and runtime construction through an outer instance do not create static inheritance edges.

## Invalid Instance Operations

A ghost in a space is ignored and produces a notice. A `super` use outside class inheritance is also ignored and produces a notice.

Typist may offer to insert an empty constructor when an edit requires a class.
