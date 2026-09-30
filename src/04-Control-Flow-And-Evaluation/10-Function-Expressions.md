# Function Expressions

Function expressions create callable values. First uses TypeScript-style arrow syntax with First type annotations.

```first
numbers.map(value => value + 1)

addFn = (a is int, b is int) => a + b
```

Use `=>` only for executable function-expression bodies. Callable types use `is` and do not contain `=>`.

```first
Adder is alias of (a is int, b is int) is int

addFn is Adder = (a, b) => a + b
```

## Parameters

A single untyped parameter may omit parentheses.

```first
names.map(name => name.length)
```

Use parentheses for zero parameters, multiple parameters, typed parameters, optional parameters, default values, rest parameters, generic parameters, and explicit result types.

```first
readFn = () => readValue()

addFn = (a is int, b is int) => a + b

chooseFn = (value is int = nextNumber()) => value

identityFn = (T is type, value is T) is T => value
```

When a parameter type is omitted, First infers it from a unique expected callable signature. If there is no unique expected signature, the parameter is `unknown`.

```first
words is string[] = ["", "hello"]
nonempty = words.filter(word => word.length > 0)
// word is inferred as string.

inspectFn = value => value.length
// value is unknown without an expected callable type.
```

Function expressions do not support parameter destructuring.

## Bodies

An expression body returns its expression.

```first
squareFn = (value is int) => value * value
```

A parenthesized single expression is still an expression body.

```first
sumFn = (a is int, b is int) => (a + b)
```

A body with declarations, control transfer, or multiple fragments is a scope body. Scope bodies require `return` to supply a value.

```first
loggedSumFn = (a is int, b is int) => (
	console.log(a)
	return a + b
)
```

An empty body is valid.

```first
emptyFn = () => ()
```

## Result Types

An explicit result type is written before `=>`.

```first
readIntFn = () is int => 1
```

When the result type is omitted, First infers it from the expression body or from reachable `return` operations targeting the function expression. If a reachable path falls through without returning, `null` is included in the inferred result and that path returns `null`.

```first
maybeOneFn = (ready is boolean) => (
	if (ready) (
		return 1
	)
)
// Inferred int or null.
```

When an explicitly typed function expression falls through, it returns the declared result type's zero value.

```first
zeroFn = () is int => ()
zeroFn() // 0
```

`declare noImplicitNullReturns` reports unannotated function expressions whose fall-through implicitly returns `null`. It does not change inference or runtime behavior.

## Callable Types

A callable type is written as `(parameters) is Result`.

```first
ReadInt is alias of () is int
Transform is alias of (value is int) is string
```

Callable result types nest to the right.

```first
Factory is alias of () is () is int
factoryFn is Factory = () => () => 42
factoryFn()() // 42
```

Name callable types with aliases before forming arrays, nullable callables, or overload-like unions.

```first
readers is ReadInt[] = []
maybeReader is ReadInt or null = null
```

Callable type extraction with `.return` and `.arguments` is defined in [[../../03-Type-System-Features/04-Aliases]].

## Defaults And Rest

Function expressions support optional parameters, default values, and a final rest parameter.

```first
connectFn = (port is int = 8080) => port
collectFn = (...values is int[]) => values
```

Default expressions run on every invocation, even when the caller supplies that argument. The supplied argument still wins; the evaluated default is discarded.

```first
chooseFn = (a is int = nextNumber(), b is int = a + 1) => b

chooseFn(10)
// nextNumber runs, a remains 10, b becomes 11.
```

Default expressions belong to the implementation, not to callable types. A type-only signature may write `= ?`, but not an executable default.

```first
Connector is alias of (port is int = ?) is int
```

## Assignment Compatibility

Callable assignment uses exact positional parameter-type matching. Result types may be covariant: an implementation may return a more specific type than the destination requires.

```first
DogReader is alias of (dog is Dog) is Animal

readDogFn = (dog is Dog) is Dog => makeDog()
readerFn is DogReader = readDogFn
// Valid: the parameter type matches and Dog is usable as Animal.

readAnimalFn = (animal is Animal) is Dog => makeDog()
badReaderFn is DogReader = readAnimalFn
// Invalid: Animal is not exactly Dog.
```

A callable may ignore trailing supplied arguments only when the destination permits those arguments and the accepted prefix types match exactly.

## Callable Unions

A union of callable aliases in callable position acts as a simple overload list. A call resolves to the first callable member in alias order whose signature accepts the provided arguments. Later, more specific signatures do not win.

```first
TextReader is alias of (value is int) is string
NumberReader is alias of (value is int) is int

Reader is alias of TextReader or NumberReader
readerFn is Reader

result = readerFn(1)
// Uses TextReader because it is first.
```

If an alias can be non-callable, call it only after narrowing to a callable type.

Callable intersections are unsupported.

## Captures

Function expressions capture immutable surrounding values lexically.

```first
offset is int = 10
addOffsetFn = (value is int) => value + offset
```

Hidden shared mutable captures are invalid for escaping function expressions. Put mutable state in an explicit field or object, or pass it as an argument.

```first
count is var int = 0
readCountFn = () => count
// Invalid if readCountFn can escape: hidden shared mutable capture.
```

A function expression may edit through an explicitly captured editable object when that object is the state container.

```first
editFn = () => (
	box.value = 3
)
```

Arrows capture their lexical receiver. Calling the stored function through another object does not rebind `this`.

```first
readNameFn = () => this.name

other.readNameFn = readNameFn
other.readNameFn()
// Reads the original lexical receiver.
```

## Function Values

Function expressions are ordinary values. They may be stored, passed, returned, and placed in containers.

```first
makeAdderFn = (offset is int) => (value is int) => value + offset
addTenFn = makeAdderFn(10)
addTenFn(2) // 12
```

Each evaluation of an arrow creates a distinct callable identity. Copying a function value preserves that identity. Callable equality uses identity; ordering is unsupported.

```first
makeFn = () => () => 1

firstFn = makeFn()
secondFn = makeFn()
copyFn = firstFn

firstFn == copyFn // true
firstFn == secondFn // false
```

Ordinary named functions may also be selected as callable values. Selecting an instance function binds the receiver.

```first
operationFn = double
operationFn(3)

readNameFn = user.makeDisplayName
readNameFn()
```

## Async, Yield, And Recovery

Async function expressions use `is async`.

```first
readAsyncFn = () is async => readRemote()
```

Function expressions cannot be generators and cannot contain function-targeting `yield`.

```first
valuesFn = () is yield int => (
	yield 1
)
// Invalid.
```

Malformed arrows do not parse. The editor should not create them. If an external tool writes one, recovery ignores the malformed line rather than synthesizing a callable fallback.

```first
brokenFn = () =>
// Invalid: missing body.
```

An explicitly empty body is different and remains valid.

```first
emptyFn = () => ()
```
