# Objects

Object literals create anonymous object shapes.

```first
myObject = { name is string = "bob", amount is int = 12 }
```

This is intentionally close to a TypeScript object literal, but the field declarations use First's ordinary `name is Type` spelling. The type is part of the literal, not a separate annotation layered on top of JSON-like syntax.

## Field Syntax

Each value-position field entry has a name, an optional type annotation, and a value.

```first
person = {
	name is string = "bob",
	age is int = 42,
	active = true
}
```

When the type is written, the value must fit that type. When the type is omitted, the field type is inferred from the value.

The commas separate fields. Field order is source order for reading and display, but the object shape is the set of named fields and their types.

In type position, the same shape is written without field values:

```first
{ name is string, age is int, active is bool }
```

## Anonymous Shapes

An object literal does not declare a class and does not create a nominal type name. It produces a value whose shape is known from its fields.

```first
total(item is { name is string, amount is int }) is int (
	return item.amount
)

value = { name is string = "bob", amount is int = 12 }
result = total(value)
```

The shape in the parameter describes the fields the function needs. A value with those fields and compatible types can be passed where that shape is expected.

Named classes remain the way to define nominal objects with constructors, functions, inheritance, startup behavior, and long-lived API boundaries.

## Access

Fields are accessed with ordinary member access.

```first
order = { id is int = 1001, amount is int = 12 }
amount = order.amount
```

An object literal only has the fields it declares. Accessing a missing field produces a notice and recovers through the ordinary `unknowable` path.

## Not JSON

Object literals are not JSON, even when they look similar.

```first
settings = {
	retries is int = 3,
	label is string = "default"
}
```

The differences are deliberate:

- field names are identifiers, not string keys
- field types can be written inline
- values are First expressions, not JSON values
- the result is a typed First value, not untyped serialized data

Use serialization APIs when a program needs to read or write JSON text.

## Mutability

Object literals follow the ordinary binding and field mutability rules. A non-`var` binding cannot be rebound.

```first
item = { name is string = "bob", amount is int = 12 }

// Rebinding, produces a notice
item = { name is string = "sue", amount is int = 14 }
```

Field-level mutability is still an open design area. Until that is specified, object literal fields should be treated as fixed after construction.

## Design Notes

Objects are for small structural values. Use a class when the value needs identity, construction rules, methods, inheritance, protected structure, or a stable public name.

The main grammar choice is that object fields use declaration syntax instead of JSON colon syntax. This keeps object literals aligned with the rest of First:

```first
name is string = "bob"
```

rather than:

```first
name: string = "bob"
```
