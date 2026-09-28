# Constructors

Constructors make a declaration a class and define how construction calls produce instances.

```first
User (
	name is string
	
	constructor(name is string) (
		this.name = name
	)
)
```

A constructor with parameters or behavior uses `constructor(...) (...)`. A constructor body may be omitted; the omitted body behaves like an empty body. A parameterless empty constructor may be shortened to `constructor`.

```first
Marker (
	constructor
)
```

Both spellings make the declaration a class. Constructors may appear in reference and primitive classes. Extension classes cannot define constructors.

## Construction

Calling a class constructs a value. First has no `new` keyword.

```first
user = User("Ada")
```

A space cannot be called. Ordinary bare declarations do not receive implicit constructors.

An inheritance clause also makes a declaration a class. When no constructor is written, an inherited class receives a synthesized constructor.

Constructor overloading is not supported. A class may have one constructor. Default and optional parameters provide multiple call shapes. The constructor may be synchronous or async.

First does not have abstract classes or constructor visibility modifiers. Inheritance parents are classes, so shared inherited structure belongs to a constructible class. A non-constructible space can group declarations but cannot serve as an inheritance parent. A nested class under a class has a protected construction path; outside code may name its type but cannot construct or inherit from it without authority over the containing class's protected route.

## Constructor Results

A constructor is a value-producing construction function whose implicit result type is the containing class. Calling `Type(...)` evaluates the constructor and yields its result.

A constructor may fall through, which returns the normally initialized `this`, or explicitly return a value assignable to the containing class type. Compatible returns include `this`, another instance of the same class, or an instance of a descendant class. Returning an incompatible value produces a notice and recovers with the typed zero value.

```first
XDataStore (
	constructor() (
		found = XDataStoreCache.current()
		if (found != null) (
			return found
		)
	)
)
```

Returning a descendant does not introduce dynamic dispatch. Static member lookup still follows the nominal type through which the value is accessed.

## Field Initialization

Every stored field must be definitely assigned by the end of construction.

```first
User (
	active is bool = true
	name is string
	
	constructor(name is string) (
		this.name = name
	)
)
```

A field type may be inferred from a definite constructor assignment. If a stored field is not definitely assigned by the end of construction, the compiler produces a notice and supplies a recovery value.

The observable `this` instance is materialized on first touch. The first touch may be explicit `this`, implicit receiver access, `super(...)`, explicit `return this`, or constructor fallthrough. At that point, First allocates and establishes the zero value of all storage, then initializes parent class parts before child class parts. Within each class body, explicit field initializers run in lexical source order, descending through instance-owned spaces where those spaces appear. Nested classes are not initialized as part of the outer instance.

First establishes storage from each field type's zero value when that zero can be synthesized. Source order is observable when one initializer reads a field whose explicit initializer has not run yet:

```first
Panel (
	In (
		seen is int = value
	)
	value is int = 7
	constructor() (
		console.log(In.seen) // 0
	)
)
```

Field-parameter assignments and required parent initialization participate in this ordinary construction process before the triggering operation proceeds.

An explicit call to a constructor with an omitted, empty, or anchor-only body still follows ordinary construction. It evaluates arguments and parameter defaults, runs field initializers, assigns field parameters, initializes required parent parts, and supplies zero values for remaining stored fields without a missing-assignment notice for that deliberate empty body.

This is distinct from synthetic zero construction used by typed fallback. Synthetic zero construction is compiler authority: it does not run user constructors, parent constructors, field initializer expressions, or startup functions. It allocates storage directly and recursively fills it with zero values. Separate zero-producing exits create separate reference instances.

If a reference type's stored fields require an impossible strong recursive construction, it has no valid synthetic zero. First reports an invalid zero-value shape rather than inventing a cyclic sentinel object.

```first
Node (
	next is strong Node
)
```

The shape becomes zero-synthesizable when it contains a legal construction break, such as `null`, weak ownership, laziness, an explicit default, or an explicit body that supplies a sanctioned value.

```first
Node (
	next is strong Node or null
)
```

Synthetic objects that bypass construction also bypass user ghosts for unconstructed class parts. Operations requiring an absent native resource produce a notice and recovery rather than forwarding fabricated handles or running cleanup for handles that were never initialized.

A path that returns another compatible instance before touching `this` skips field initialization for the abandoned would-be instance. A path that returns another compatible instance after touching `this` abandons the materialized partial instance and uses ordinary incomplete-construction cleanup for its initialized fields and parts.

Constructors may bomb. A constructor body may `throw`, or use the postfix `throw` operator to pass through a bomb from another expression. When a constructor can bomb, calling the class produces `ClassName | bomb` and the call site must handle or propagate the bomb like any other bombable expression.

```first
User (
	name is string
	
	constructor(name is string) (
		if (name == "") (
			throw "User name is required"
		)
		
		this.name = name
	)
)

user = User(inputName) 💣 throw
```

On a bombing path, construction produces no usable instance. If `this` has not materialized yet, there is no instance cleanup. If `this` has materialized, ordinary incomplete-construction cleanup applies to the initialized fields and parts. Definite-assignment requirements apply to successful construction paths.

## Field Parameters

A constructor parameter may declare and initialize a same-named stored field by writing `is field of T`.

```first
Document (
	status is DocumentStatus

	constructor(
		identifier is field of string
		accountName is field of string) (
		this.status = DocumentStatus.open()
	)
)
```

The parameter value is assigned to the stored field when `this` materializes. Field parameters replace explicit constructor copies for their stored fields.

```first
Document (
	identifier is string
	accountName is string
	status is DocumentStatus

	constructor(identifier is string, accountName is string) (
		this.identifier = identifier
		this.accountName = accountName
		this.status = DocumentStatus.open()
	)
)
```

The field type is part of the field parameter. The field does not need a separate declaration.

```first
ScrapedPage (
	constructor(
		target is field of ScrapeTarget
		url is field of string
		html is field of string
		capturedAt is field of i64)
)
```

Field parameters participate in ordinary construction. They satisfy definite assignment for their stored fields on successful fallthrough or `return this`, and any remaining stored fields must still be definitely assigned by the end of the constructor. A constructor path that returns another compatible instance before `this` materializes does not assign field parameters to an abandoned instance.

## Super Calls

When a class initializes its own instance and inherits from a parent with a constructor, its constructor must call `super(...)`.

```first
Animal (
	constructor(name is string) (
		this.name = name
	)
)

Bunny is Animal (
	constructor(name is string) (
		super(name)
	)
)
```

Typist inserts required `super(...)` calls automatically on paths that materialize `this`. These calls are fixed tokens: they may be reordered where permitted but cannot be deleted while the path still initializes `this`. `this` is unavailable until every required parent constructor call has completed.

When a child constructor omits its body, implicit parent calls are available only when every required parent argument can be supplied by defaults or optional parameters. Missing required parent arguments produce a notice and construction recovery until an explicit body supplies the call. First does not silently zero required parent arguments.

If a constructor returns another compatible instance before `this` materializes, the constructor does not need to initialize parent parts for an abandoned instance.

Inside `super(...)`, a parent constructor must initialize the child's parent part. A parent constructor path that tries to replace the child instance by returning a different object through `super(...)` produces a notice and construction recovery; First does not copy cached objects or resources into the child.

With multiple inheritance, the constructor calls one parent constructor per parent. Parent order determines the default call order. Incompatible inherited initialization produces a notice.

Returning from `super(...)` means the parent's fields and constructor body are complete. It does not run startup blocks from the parent's instance-owned spaces. After all required parent constructor bodies and the most-derived constructor body complete, inherited startup runs in parent order and local startup follows. Construction returns its value only after that traversal succeeds.

A `super` use outside a valid instance inheritance context is ignored and produces a notice.

## Async Constructors

A constructor may be asynchronous.

```first
Foo (
	constructor(initialValue is int) is async (
		this.value = await load(initialValue)
	)
	
	value is int
)
```

Calling a class with an async constructor produces a promise resolving to the constructor result. If the async constructor can bomb, the promise resolves to `ClassName or bomb`. Async constructors support field parameters, field assignment, inherited-constructor behavior, explicit compatible returns, and ordinary bomb propagation.

For TypeScript output, async construction lowers to a static async `new()` method. Rust output uses an async associated construction function. A class with an async constructor cannot expose a space member that would also emit as `new`.

## Initialization Outside Instances

Spaces use startup functions for initialization.

```first
Registry (
	items is Array(string) = []
	
	startup (
		items.push("ready")
	)
)
```

Program-owned startup follows the ordinary startup-order rules. A space below a class instead contributes startup to construction after constructor bodies complete. Space-member access does not trigger hidden first-use initialization.
