Properties define stored or computed member access on spaces and classes.

A property is written as a getter, a setter, or both.

```first
User (
	constructor
	
	get name() is string (
		return firstName + " " + lastName
	)
	
	set name(value is string) (
		firstName = value
	)
)
```

This spelling intentionally stays close to TypeScript accessor syntax. First uses `is` for type annotations and `(` / `)` for bodies.

## Accessors

A getter is written with `get`, the property name, an empty parameter list, a return type, and an optional body.

```first
get name() is string (
	return firstName + " " + lastName
)
```

A setter is written with `set`, the property name, one parameter, and an optional body.

```first
set name(value is string) (
	firstName = value
)
```

A property may have only a getter, only a setter, or both. A space or class may define at most one getter and one setter for the same property name.

Setters do not return values.

A bodyless getter with no declared setter exposes a stored value for reading. Code within the owning class or space can assign that value; code outside the owner cannot assign it without a declared setter. Until assigned, the stored value has its type's ordinary zero value.

```first
User (
	constructor(name is string) (
		this.name = name
	)
	get name() is string
)

user = User("Ada")
console.log(user.name)
// > "Ada"
user.name = "Grace"
// > Notice: no externally accessible setter.
```

Contributions in different partitions retain the same logical owner. An implementation contribution can assign the value when the property is visible under the ordinary partition rules: its declaration must be in the contribution's own partition or an ancestor. A different partition does not by itself make the contribution external to the class or space. See [[18-Partitions]].

A getter with an executable body computes its result using ordinary return and fall-through rules.

## Getter And Setter Types

When a property has both a getter and a setter, the getter return type and setter parameter type may differ.

First should follow TypeScript's accessor behavior as closely as possible while preserving First's nominal type system. Where TypeScript relies on structural assignability, First uses nominal compatibility.

```first
Setting (
	constructor
	
	get value() is SavedValue (
		return savedValue
	)
	
	set value(value is NewValue) (
		savedValue = value.toSavedValue()
	)
)
```

## Access

Property reads and declared setters follow ordinary structural access. A direct property on a class is public. A property inside a nested space under a class is protected through that nested path. A bodyless getter alone does not expose assignment outside its owner.

```first
User (
	constructor
	In (
		get id() is string (
			return internalId
		)
	)
	
	get id() is string (
		return In.id
	)
)
```

`user.id` is public. `user.In.id` produces a notice outside `User`.

## Space Properties

Properties may appear in spaces.

```first
Registry (
	get count() is int (
		return items.length
	)
)
```

Space properties do not have an implicit instance.
