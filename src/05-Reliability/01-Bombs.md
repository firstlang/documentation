In First, a bomb is a special kind of error value that propagates through expressions without interrupting control flow, unlike traditional exceptions. Instead of forcing you to handle failures immediately, bombs allow you to pass them up the call stack easily.

Editors will render the 💣 glyph after all identifiers that are contaminated with bombs. This way, the author is incentivized to deal with it right away, and avoid passing them around the program.

If a function might fail, we use First's partial inference feature to convert the function into returning a value of type `T | bomb` (instead of just `T`). This approach eliminates the need for try/catch while making failures visible and intuitive—without getting in your way.

# Throw As Statement

The most natural way to throw a bomb is to just use a `throw` expression that TypeScript developers are already familiar with:

```first
// This method has a throw, so the 💣 appears,
// and the return type of the method becomes int | bomb
// instead of just int.
start() is int 💣 (
	if (1 + 1 === 3) (
		throw "We can't math."
	)
	
	return 0
)
```


# Automatic Bombs

You don't need to provide a bomb value. You can just throw without a value and it will create a message for you.

```first
start() (
	if (1 + 1 === 3) (
		throw // Bomb message is "Failure in start(), at SourceFile.first:123"
	)
	
	return 0
)
```

When you call a function that could possibly return a bomb, you can't touch the result. Instead, you need to deal with the bomb somehow. Here is the more manual way of doing that:

```first
start() (
	🥚 foo = getFooOrBomb() 💣 // foo is Foo | bomb
	if (foo is not bomb) (
		console.log(foo.bar)
	)
)
```

# Throw As Operator

Alternatively, you can use `throw` as an operator and suffix it after the bomb-ridden expression. Using throw as an operator is the quick escape hatch to pass it off to the containing context without thinking too much.

```first
// start() is bombable now, because of the throw operator.
start() 💣 (
	🥚 foo = getFooOrBomb() 💣 throw // foo is just Foo
	console.log(foo.bar) // Works
)

iterate() (
	// v1 is of type (Foo | bomb)[]
	🥚 v1 = 1 to 10 each i (
		// throw is pointless here, you're already yielding Foo | bomb
		// This generates a compliation notice.
		yield getFooOrBomb() 💣 throw
	)
	
	// v2 is of type (Foo | bomb)[]
	🥚 v2 = 1 to 10 each i (
		// You could do this, which is idiomatic
		🥚 fooOrBomb = getFooOrBomb() 💣 throw
		
		// You could also do this which is a lot less idiomatic,
		// but we might have to also support this:
		🥚 fooOrBomb = getFooOrBomb() 💣
		
		// Perhaps the bomb glyph keeps following bombed identifiers. 
		// This will discourage this is gross pattern below, which
		// technically would need to work. 
		fooOrBomb 💣 throw
		
		// This would actually be executed below fooOrBomb throw;
		// in the case when fooOrBomb is not a bomb
		yield fooOrBomb
	)
)

```

When `throw` is used as an operator, it's placed as a postfix-style suffix behind a bomb-throwing expression. It therefore has high precedence order. For example:

```first
// Given
foo = a() 💣 throw + b()
// This means:
foo = (a() 💣 throw) + b()
```

Because the throw operator always proceeds a 💣 glyph, the Tokens editor will render this as a compound token in order to re-enforce that the bomb was generated, and is escaping out.

# Throw As Expression

Throw as expression is a convenient way to get an early quit mid-expression if something isn't going as planned. This is part of the effort to maximize ergonomics and reduce friction around immutability.

```first
start() (
	🥚 str = getStringMaybe() 💣
	if (str is bomb) (
		console.log("A bomb happened") // Logs "Bomb"
	)
	else (
		console.log("[" + str + "]") // Works because we know its a string
	)
	
	🥚 bar = getStringMaybe() 💣
	if (bar is bomb) (
		console.log(bar) // Logs the auto-generated bomb message
	)
)

getStringMaybe() is string 💣 (
	return Math.random() > 0.5 ? "Hello" : throw "Bomb"
)

getStringOrGenericBomb() is string 💣 (
	return Math.random() > 0.5 ? "Foo" : throw
)
```

## Bomb Propagation

Propagation happens using a bomb type inference system.

```first
mightFail() is string 💣 (
	if (Math.random() > 0.5) (
		throw "Error"
	)
	else (
		return "success"
	)
)

propagate1() 💣 (
	🥚 value = mightFail() 💣
	return value
)

propagate2() 💣 (
	🥚 value = propagate1() 💣
	return value
)
```

# Catching Bombs

You can catch bombs using the `catch` operator. It is worth noting that `catch` in First operates nothing like `catch` in most try/catch supporting languages. Instead it's conceptually much closer to the nullish coalescence operator (`??`).

The catch operator essentially does "expression fall-through, if and only if the value is a bomb". For example:

```first
handle() (
	🥚 str = getStringMaybe() 💣 catch "Bomb"
	return str + "!" // Returns Hello! or Bomb!
)

getStringMaybe(): string 💣 (
	return Math.random() > 0.5 ? "Hello" : throw "Bomb"
)
```

Below is a more complex example using `catch` with the nullish coalesce operator, to show how the fall through operates:

```first

nullishFirst() (
	🥚 str = getStringNullBomb() 💣 ?? "Null" catch "Bomb"
	return str + "!" // Returns Hello! or Null! or Bomb!
)

catchFirst() (
	🥚 str = getStringNullBomb() 💣 catch "Bomb" ?? "Null"
	return str + "!" // Returns Hello! or Bomb! or Null!
)

getStringNullBomb() is string 💣 (
	🥚 r = Math.random()
	if (r < 0.33)
		return "Hello"
	
	if (r < 0.66)
		return null
	
	throw "Bomb"
)
```

# Throwable Values

The author may choose to throw either a string, a `bomb` object, or nothing at all. Each are reasonable. The value is converted into a `bomb` object regardless.

# Bomb Structure

`bomb` is a first-class object in First. It's structure is created in-place. The structure of a bomb is defined like so:

```first
📦 Bomb (
	message        is int;
	code           is int;
	sourceFile     is string;
	sourceFunction is string;
	sourceLine     is int;
)
```

Bombs can be inherited from if the author desires to carry custom data within the bomb.

# Throw Expressions

Throw is an expression. You can put it in places that take an expression, though it terminates the execution.

The "throw" keyword is an expression in First. This makes handling bomb propagation much more natural.
This is important, but there are design issues. For example, in this case, its obvious that foo() should be a bombable function, but how does it know this?

```first
foo() (
	// It's actually a compiler error to throw in here, because the callback function
	// does not explicitly allow it, because it's not bombable. Which means that if
	// you actually do want your closures to be able to throw bombs, you have to
	// explicitly declare that. I think this means that we don't need library functions
	// that support early-termination on iteration.
	// Are you sure this doesn't work? Wouldn't it not just infer that the return type
	// of the map operation is bombable, and then const item just becomes bombable?
	// I don't see an issue here, a bomb is basically just a symbol
	🥚 item = items.map(x => x.valid ? x : throw "Invalid item")💣
)
```

How would it make this terminate the map operation? Because that is what would need to happen.

I think you need to do more studying here to get a better sense of what the call pattern looks like for zig's error propagation (which is basically what we're trying to model) and TypeScripts try/catch

For certain common operations like Array#map, Array#find, Array#filter, you might be able to just create inline variations of the loop. This might require some elaborate code rewriting, but it would eliminate the closure usage.

A more realistic option would probably be to have a custom library that supports Interruptible array operations, which would compile to something like this:

```first
foo() (
	// T_.Array.map would support interruptible iteration, 
	// which checks to see if the returned value is an error and quits out.
	// You would do hindley-milner bomb inference to see if the Array$map
	// method would be needed, and only use it when there is a possibility
	// when the closure could return a bomb.
	🥚 item = T_.Array$map(items, x => x.valid ? x : T_.bomb("Invalid item");
	
	// Note that this doesn't solve the problem generally.
)
```
