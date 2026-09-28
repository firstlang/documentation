Extension classes group functions that can be called as members on an existing type.

They are useful when behavior belongs with a type at the call site, but the original type should not be structurally reopened or cannot be reopened because it comes from an imported package.

```first
🔌 StringMethods is extension of string (
	reverseTrim() is string (
	)
	
	trimUnicode() is string (
	)
)
```

An extension class is not a class in the nominal type sense. It does not create values, does not participate in inheritance, and does not add nominal weight to the type it extends. It is a transparent grouping construct for extension members.

## Targets

An extension class can target primitives, reference classes, and primitive classes.

```first
🔌 StringMethods is extension of string (
)

🔌 UserMethods is extension of User (
)

🔌 PointMethods is extension of Point (
)
```

Extension classes may target local types, imported package types, primitives, primitive groups, and other target families accepted by the type system. They do not structurally merge into the target. They attach callable behavior and eligible operator overloads while leaving the target's owned declaration unchanged.

First does not have a capitalized `String` object type. `string` is the string type.

## Primitive Groups

Some primitive families are commonly extended together. The standard library provides a series of endowed primitive groups (defined in [[03-Numbers]]) that can be extended.

```first
🔌 IntegerFormatting is extension of AnyInteger (
	
	toHexString() is string (
	)
)
```

These groups are special standard library targets. They stand for a set of primitive types and avoid requiring the same extension to be written separately for each primitive.

## Multiple Targets

An extension class can target multiple types with a comma-separated target list.

```first
🔌 ReadableTools is extension of string, File, Buffer (
	
	readText() is string (
	)
)
```

This is repeated application of the same extension body to each target. It is not multiple inheritance.

When an extension applies to multiple targets, `this` has the union of the possible target types.

```first
🔌 DebugTools is extension of string, User (
	
	debugLabel() is string (
		return this.toString()
	)
)
```

## This

Inside an extension member, `this` refers to the value being extended.

```first
🔌 StringMethods is extension of string (
	
	wrapped() is string (
		return "[" + this + "]"
	)
)
```

For primitive targets, `this` is passed by value. Primitive extension members cannot mutate the primitive value; they return a new value instead.

For class targets, an extension member can mutate public mutable properties on `this`, but this is discouraged. Extension methods should usually behave like ordinary helper behavior rather than hidden mutation.

## Members

Extension classes can contain functions, getters, setters, and operator overload functions.

They cannot contain fields, constructors, ghosts, nested spaces, or nested classes.

An extension operator overload is only allowed when the operator has not already been overloaded for the target.

## Lookup

When resolving a member call, real members on the type win over extension members.

```first
User (
	constructor
	
	displayName() is string (
		return "real"
	)
)

🔌 UserDisplayExtensions is extension of User (
	displayName() is string (
		return "extension"
	)
)
```

In this example, `user.displayName()` calls the real `User` function. Extension classes cannot overwrite real functions.

If more than one applicable extension provides the same member, ordinary member lookup keeps real members first. Extension conflict and shadowing rules are under review; the compiler must not silently choose by file order.

```first
🔌 FirstUserExtensions is extension of User (
	label() is string (
		return "first"
	)
)

🔌 SecondUserExtensions is extension of User (
	label() is string (
		return "second"
	)
)
```

In this example, `user.label()` needs the extension conflict rules to choose one extension or report a notice.

When a program needs a specific extension member instead of the inferred one, it can attest the access path.

```first
(user is FirstUserExtensions).label()
```

The attestation selects the extension member for lookup. It does not cast the value into an extension class, because extension classes do not exist as runtime or nominal types.

## Structural Availability

An extension class is a structural declaration. Its availability follows the program's structural and import graph rather than source-file placement.

An extension defined in the current program participates in the effective structural program. An extension provided by an imported package participates according to the package import rules. Source files do not create local extension scopes.

An extension target does not grant protected access to the target's nested implementation partitions. Extension bodies have the authority of their own logical declaration, not the authority of the type they extend.

```first
Panel (
	constructor
	In (
		count is int = 7
	)
)

PanelTools is extension of Panel (
	readCount() is int (
		return this.In.count
	)
)
// Notice: extending Panel does not grant Panel's protected authority.
```

## Inheritance

Extension classes do not participate in inheritance.

If an extension targets a base class, subclasses can use that extension through normal subtype compatibility.

```first
Animal (
	constructor
	
	eat() (
	)
)

Dog is Animal (
)

🔌 AnimalExtensions is extension of Animal (
	sleep() (
	)
)

dog = Dog()
dog.sleep()
```

If the subclass already has a real member with the same name, the real member blocks the extension member.

Extensions cannot override inherited members. They also cannot call `super`.

To call a member from a compatible base view, attest `this` to that base type.

```first
Animal (
	constructor
	
	makeSound() (
	)
)

Dog is Animal (
	makeSound() (
	)
)

🔌 DogExtensions is extension of Dog (
	makeAnimalSound() (
		(this is Animal).makeSound()
	)
)
```

## Packages

Any package can define extension classes for any type it can name, including types from another package. Those extensions become available only through the ordinary package/import graph; packages do not globally patch foreign types merely by existing.

## Limits

Generic extension classes are not supported.

Constrained extension classes are not supported.

Extension classes cannot be instantiated, inherited from, inherited by, or used as ordinary nominal types.
