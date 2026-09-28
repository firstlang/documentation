Startup functions define code that runs automatically for the storage owner of their containing space. In a program-owned space they run when the program starts. In a space owned by a class instance they extend that instance's construction.

A startup function is function-like, but it is not a normal callable function. It has a body, can use the same statements as a function body, and follows the same lexical scope rules. It does not have a name, parameters, or a return type.

```first
startup (
	console.log("Starting")
)
```

First uses startup functions instead of requiring a special `main()` function.

## No Parameters

Startup functions cannot take parameters.

Command-line arguments are accessed through the runtime API instead of being passed into the startup function.

```first
startup (
	console.log(process.argv)
)
```

This is invalid:

```first
startup(args is string[]) (
)
```

## No Return Type

Startup functions do not declare a return type.

```first
startup (
	setup()
)
```

Any value produced inside a startup-function body is ignored unless the program uses it explicitly.

This is invalid:

```first
startup is int (
	return 1
)
```

## Not Callable

A startup function is called by the program runtime. User code cannot call it directly.

This is invalid:

```first
startup()
```

If code needs to be shared between a startup function and another part of the program, put that code in a normal function and call the function from both places.

```first
setupLogging() (
)

startup (
	setupLogging()
)
```

## Placement

Startup functions can appear at the top level or inside spaces at any depth.

```first
startup (
	loadConfig()
)

Server (
	startup (
		listen()
	)
)
```

Startup functions cannot appear directly inside classes. A space inside a class may contain startup because that space groups members of the surrounding instance.

This is valid:

```first
Server (
	constructor
	In (
		startup (
			prepare()
		)
	)
)
```

This remains invalid because the startup is a direct class member:

```first
Server (
	constructor
	startup (
	)
)
```

## Multiple Startup Functions

A program can contain more than one startup function.

```first
startup (
	configureLogging()
)

startup (
	connectDatabase()
)
```

Program-owned startup runs after imported wrapper startup has completed. For each storage owner, startup blocks run in lexical source order using a depth-first traversal of its spaces. A block runs where it appears; when traversal reaches a nested space, that space is traversed before later declarations in the enclosing space. Nested classes are not traversed, because each nested class has its own construction lifecycle.

```first
Panel (
	constructor
	Alpha (
		startup (
			console.log("alpha 1")
		)
		Inner (
			startup (
				console.log("inner")
			)
		)
		startup (
			console.log("alpha 2")
		)
	)
	Beta (
		startup (
			console.log("beta")
		)
	)
)

panel = Panel()
// alpha 1, inner, alpha 2, beta
```

This lets setup code live beside the space that owns it.

## Instance Construction

Instance-owned startup runs only after all required parent constructor bodies and the most-derived constructor body have completed successfully. A `super(...)` call initializes the parent fields and runs the parent constructor body; it does not run parent-owned startup before returning.

Startup remains inside the construction success boundary. The instance is unavailable to its caller until every startup block completes. A bomb stops the remaining traversal and fails construction through ordinary constructor bomb handling. An `await` inside startup is permitted only when the owning constructor is declared `constructor(...) is async`; startup has no separate async annotation.

Inherited startup contributions all run. Parent class parts are traversed in parent order, each using its own source-order depth-first traversal, followed by the child's local traversal. Same-name merged spaces do not override one another's startup blocks. Ordinary later-parent precedence still governs member lookup after construction.

## Wrapper Startup

Imported wrapper packages may provide generated startup functions. These functions are distinct from project startup functions: they are wrapper metadata used to make an imported module ready before its API is used.

Every valid imported wrapper participates in startup, including wrappers imported inside unused spaces or classes and wrappers imported by test targets. Each package version starts at most once.

Wrapper startup functions must be synchronous. They cannot use `await`, return bombs, or require caller-managed failure handling. A module requiring asynchronous or fallible initialization must expose an ordinary function that project startup-function code calls explicitly.

Startup functions should be independent. If static analysis finds that one startup function accesses another imported module, the compiler records that dependency and topologically orders the generated calls. Independent functions have no semantic order. A dependency cycle produces notices and short-circuits every startup function in the cycle through `unknowable` recovery.

The compiler injects the generated wrapper startup dispatcher into Rust `main`. It completes before ordinary project startup functions run.

If a wrapper startup function attempts to return a bomb, the compiler produces a notice. If a bomb occurs at runtime, execution of that startup function stops.

## Tests

Program-owned startup functions are not called when the entry point is a test.

Tests should set up the program state they need directly. Constructing a class during a test still runs its instance-owned startup as part of ordinary construction.

Imported wrapper startup functions still run for tests. They are part of making imported modules valid, not application-specific project startup.
