Properties define computed member access on spaces and classes.

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

An omitted accessor body behaves like an empty accessor body. A getter with an explicit result type returns that type's zero value on fall-through. A setter with no body evaluates the receiver and assigned value, then performs no stored update unless an implementation partition supplies one.

```first
get name() is string
set name(value is string)
```

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

Properties follow ordinary structural access. A direct property on a class is public. A property inside a nested space under a class is protected through that nested path.

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

## Automatic Properties

First does not currently have a condensed automatic property syntax.

Fields, properties, and explicit constructor assignments cover the supported storage and API shapes. The text format should not add a second spelling only to make common read/write access shorter.
