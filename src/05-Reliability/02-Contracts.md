Contracts are runtime checks written with the `ensure` keyword. They make assumptions and guarantees explicit in executable code, and they give the compiler, interpreter, debugger, and editor a deterministic mechanism to reason about narrowed values.

An `ensure` statement fails if its expression evaluates to `false` or `null`.

```first
divide(value is int, divisor is int) (
	ensure divisor != 0
	
	return value / divisor
)
```

Contracts are language syntax rather than ordinary function calls. This lets release builds strip them, lets the compiler use them for narrowing, and lets postconditions reference the special `return` value.

# Preconditions

An `ensure` that does not reference `return` runs immediately when execution reaches it.

```first
read(filePath is string) (
	fileText = Fs.readFile(filePath) 💣
	ensure fileText is not bomb
	return fileText
)
```

After `ensure fileText is not bomb`, `fileText` is narrowed from `string | bomb` to `string`.

Contracts can appear inside ordinary scopes. A narrowing created by an `ensure` applies after the statement within the same control-flow path.

```first
handle(value is Foo or Bar) (
	ensure value is Foo
	
	return value.doFooThing()
)
```

# Postconditions

An `ensure` that references `return` is a postcondition. It checks the value currently being returned.

```first
normalize(value is string) (
	result = value.trim()
	return result
	
	ensure return.length > 0
)
```

Postconditions follow normal source order. A postcondition cannot reference `return` before a return value exists.

```first
normalize(value is string) (
	ensure return.length > 0 // Notice: no return value exists yet.
	
	result = value.trim()
	return result
)
```

This avoids hidden defer-like behavior. Returning does not register earlier code to run later. Instead, after a `return`, execution continues forward through applicable postconditions until the current scope has been exited.

# Scoped Postconditions

Postconditions belong to the scope where they appear. When a `return` exits nested scopes, execution continues through the remaining postconditions on that path before the function completes.

```first
process(value is Foo or Bar) (
	if value is Foo (
		return value
		
		ensure return.fooField > 0
	)
	
	ensure return.isValid()
)
```

If the `Foo` branch returns, the branch postcondition runs first, followed by the outer postcondition.

Postconditions in scopes that were not entered do not run.

```first
process(value is Foo or Bar) (
	if value is Foo (
		return value
		
		ensure return.fooField > 0
	)
	
	if value is Bar (
		return value
		
		ensure return.barField != null
	)
)
```

Returning from the `Foo` branch does not run the `Bar` branch postcondition.

# Narrowed Return Values

Inside a scoped postcondition, `return` has the type known for that return path at that point in the scope.

```first
process(value is Foo or Bar) (
	if value is Foo (
		return value
		
		ensure return.fooField > 0
	)
)
```

The postcondition can access `return.fooField` because the branch narrowed `value` to `Foo`, and that branch returned the narrowed value.

Outer scopes see `return` through the type information available in the outer scope.

# Compile-Time Narrowing

Some `ensure` checks can affect compile-time analysis even though contracts are runtime checks.

```first
getItem() (
	item = getItemOrNone()
	ensure item != none
	
	return item
)
```

The compiler may use the contract to narrow away `none` after the `ensure`.

# Future Refinement Types

Contracts are not refinement types yet. They are runtime checks with limited compile-time narrowing.

The syntax is intentionally compatible with a future where contracts may participate in deeper static analysis.
