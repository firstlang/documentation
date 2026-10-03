# Class-Level Constructs

An ordinary class body may contain fields, functions, properties, a constructor, a ghost, imports, nested spaces, and nested classes. Imports precede all other semantic members. A class has at most one constructor and one local ghost.

```first
Panel (
	import Theme

	constructor(title is string) (
		this.title = title
	)

	title is string

	In (
		count is var int = 0
		startup (
			count += 1
		)
	)

	Item (
		constructor
	)
)
```

A startup function is not a direct class member. It may occur in a nested space, where it participates in construction of the owning class instance. A nested space does not create an object or receiver; its fields and functions remain members of the surrounding instance under their qualified paths.

A nested space is a protected structural partition of the class. A nested class creates a protected nominal type and a new instance boundary. Its declaration-qualified path is valid in type positions. Calling it requires authority over its owning class:

```first
panel = Panel("Inbox")
item is Panel.Item = panel.Item()
```

Extension classes, primitive classes, and selection types have their own member restrictions. Their dedicated reference pages override this ordinary class-body list.

Stored fields may use an anchor-only body after their complete declaration. The body owns anchors for the field and does not create a runtime scope or executable startup location.

The `is Type` portion of a field declaration is optional. A bare field is equivalent to `is unknown`. When a field has an initializer and no explicit type, the initializer determines the field type. Eager field initializers are restricted to literals and single named values; expression evaluation belongs in constructors, lazy field initializers, or other executable bodies.

```first
Panel (
	identifier
	mode = DefaultMode
	title is string (
		// Navigation
		- Names the panel for visible navigation.
	)
)
```
