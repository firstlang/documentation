# Visibility

First does not use visibility modifiers or visibility lists for ordinary source declarations. Access follows the program's structural hierarchy.

Direct ordinary members of a class are public. Nested spaces and nested classes inside a class are protected structural partitions. They are visible to the owning class, structurally nested class bodies when they have an explicit compatible receiver, and nominal descendants. They are not public instance paths.

```first
Panel (
	constructor
	In (
		count is var int = 0
	)
	count() is int (
		return In.count
	)
)

panel = Panel()
console.log(panel.count()) // Valid.
// panel.In.count: notice.
```

`In` is only a convention for an internal partition. It is not a keyword and does not receive special treatment. Any nested space below a class has the same protected access rule.

## Nested Types

A protected nested class may be named in type positions, return types, generic arguments, and ordinary type comparisons. Naming the type does not grant construction or protected member access.

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

show(item is Panel.Item) (
	console.log(item.name)
)

panel = Panel()
show(panel.createItem()) // Valid.
// panel.Item(): notice outside Panel.
```

Using a protected nested class as an inheritance parent requires the same protected authority as constructing it. Code may name a returned nested type without being allowed to derive from it.

## Descendants

A descendant class may access protected partitions owned by its ancestors. The access check is on the declaring body, not on whether the receiver is the current object.

```first
Base (
	constructor
	In (
		count is int = 7
	)
)

Child is Base (
	read(other is Base) is int (
		return other.In.count
	)
)
```

An inherited parent body keeps the authority it had where it was declared. It does not gain access to child-only partitions or unrelated-parent partitions merely because a child merges inherited spaces.

## Structural Fragments

Every accepted fragment of a local class contributes to that class and has that class's access rights. Protection is not meant to hide a class from source that is allowed to become part of that class.

Imported package declarations retain package identity and cannot be reopened merely by writing a matching path in consuming code. Extensions may target imported types, but an extension target does not grant protected authority over the target.

## Invalid Access

Attempting to cross a protected structural partition without authority produces a notice. The protected access does not execute; the affected operation becomes a no-op under ordinary recovery.

```first
Panel (
	constructor
	In (
		count is int = 7
	)
)

panel = Panel()
console.log(panel.In.count)
// Notice at In; the statement does not run.
```

Legacy visibility-list syntax is obsolete. Declarations such as public API members should be written as direct members, and internal responsibility groups should be nested spaces.
