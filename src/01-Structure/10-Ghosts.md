Ghosts define destructor behavior for class values.

A ghost is written with the `ghost` keyword followed by a body. It has no name, parameters, return type, generic parameters, or overload list.

```first
FileHandle (
	handle is NativeHandle
	
	constructor(path is string) (
		this.handle = open(path)
	)
	
	👻 ghost (
		this.handle.close()
	)
)
```

The editor renders the 👻 unicode character before a ghost declaration when one is detected. The stored source keyword is `ghost`.

Ghosts are First's spelling for Rust destructors. For Rust output, a ghost lowers to an implementation of `Drop::drop`.

## Placement

Reference classes may define ghosts.

Primitive classes cannot define ghosts because primitive classes are immutable value types and do not have object identity.

Extension classes cannot define ghosts because extension classes do not create values.

Spaces cannot define ghosts. Program-owned fields follow the backend's normal static lifetime behavior. Fields grouped in an instance-owned space are destroyed as fields of the owning class.

A class may define at most one local ghost.

## This

Inside a ghost, `this` refers to the value being destroyed.

```first
Subscription (
	constructor
	
	id is string
	
	👻 ghost (
		Registry.unsubscribe(this.id)
	)
)
```

`this` is a mutable reference to the still-valid object. Stored fields and ordinary instance members may be accessed through `this`.

A ghost does not receive ownership of `this`. It may clean up resources owned by the object, but it cannot move fields out of the object.

If a field value must be taken during destruction, store it in an optional or other replaceable container and leave the field in a valid replacement state. This mirrors Rust's rule that `Drop::drop` receives `&mut self`, not `self`.

## Timing

A ghost runs automatically when the object is destroyed.

For owned values, destruction happens when the value's lifetime ends. For reference-counted fallback values, destruction happens when the final strong reference is released. The ownership system determines which storage strategy is used; the ghost semantics are the same either way.

Ghosts are not called directly.

```first
user.ghost() // notice
```

Ghost execution is deterministic for ordinary owned values and follows the same lifetime points used by Rust. For reference-counted values, the last-reference point determines when the ghost runs.

## Destruction Order

When a value is destroyed, the most-derived local ghost body runs before stored fields are destroyed.

After the ghost body finishes, stored fields are destroyed automatically in the reverse of their initialization order. Instance-owned spaces add no destruction stage: their fields occupy their source-order positions in the owning class's field sequence.

```first
Pair (
	constructor
	
	first is Resource
	second is Resource
	
	👻 ghost (
		log("pair")
	)
)
```

In this example, destruction runs the `Pair` ghost, then destroys `second`, then destroys `first`.

This matches Rust's destructor shape: custom destructor code runs first, and then fields are dropped automatically.

## Inheritance

Ghosts are inherited like other class members.

Because First inheritance behaves like structural embedding with nominal history, inherited ghosts are treated as destructor parts of the child value.

For a class with parents, destruction runs in the reverse of construction order:

1. The child's local ghost runs first, if present.
2. Inherited parent ghosts run in reverse parent order.
3. Fields are destroyed in reverse declaration order for each corresponding class part.

```first
Animal (
	constructor
	
	👻 ghost (
		log("animal")
	)
)

Pet (
	constructor
	
	👻 ghost (
		log("pet")
	)
)

Bunny is Animal, Pet (
	👻 ghost (
		log("bunny")
	)
)
```

Destroying a `Bunny` logs:

```first
"bunny"
"pet"
"animal"
```

Parent order is the order written in the class declaration. Since constructors run in parent order by default, ghosts run in the reverse order.

Inherited duplicate ghosts are allowed. Each inherited class part is destroyed once.

## Super

A ghost does not call `super`.

The compiler is responsible for running inherited ghosts and field destruction in the correct order. User code should only describe the cleanup owned by the current class part.

```first
Bunny is Animal (
	👻 ghost (
		cleanUpBunny()
		// no super.ghost()
	)
)
```

## Partial Construction

If construction does not complete, only fully initialized class parts and fields are destroyed.

This mirrors Rust's partial-initialization behavior: values that have been successfully initialized are cleaned up, and values that were never initialized are not destroyed.

For a class with inherited constructors, a parent class part is considered initialized after its required `super(...)` call completes.

## Async

Ghosts cannot be marked `async` and cannot use `await`.

Destructor work must be synchronous. If cleanup requires async behavior, the class should expose an explicit async close, dispose, or finish function. A ghost may perform fallback synchronous cleanup for forgotten explicit cleanup, but it must not depend on async execution.

## Notices

The compiler generates a notice when:

- A class declares more than one local ghost.
- A ghost is declared in a space, primitive class, or extension class.
- A ghost declares parameters, a return type, generic parameters, or overload variants.
- A ghost is called directly.
- A ghost attempts to move a stored field out of `this`.
- A ghost is marked `async` or uses `await`.

As a recovery fallback for duplicate local ghosts, the last local ghost wins for code generation. Earlier duplicate ghosts remain in the source and are reported by notices.

## Failure

A ghost should not fail.

If a ghost bombs or otherwise exits abnormally, backend behavior follows Rust destructor behavior as closely as possible. During normal execution, the failure propagates according to the surrounding backend's panic or exception model. During stack unwinding, a second failure from a ghost may abort the program.

First code should treat ghost failure as a last-resort programming fault rather than a normal control-flow path.

## Rust Output

For Rust output, a local ghost lowers to `Drop::drop`.

```first
FileHandle (
	constructor
	
	handle is NativeHandle
	
	👻 ghost (
		this.handle.close()
	)
)
```

Equivalent Rust shape:

```rust
impl Drop for FileHandle
{
	fn drop(&mut self)
	{
		self.handle.close();
	}
}
```

The generated Rust code must preserve First's class-part destruction order while still relying on Rust's ordinary field drop behavior wherever possible.
