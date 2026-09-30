The generics in First are inspired by TypeScript, but with a reduced feature set in order to work towards the language's goals of maximizing intelligibility, as well as to allow the code to more naturally be translated into Rust.

This document is disorganized—parts of it are being moved into aliases and attestations. See [[04-Aliases]] for the public alias rules and callable type extraction.

---

## Type Aliases

In First, **type aliases** are always defined using the `alias of` compound keyword:

```
Password is alias of string
```

This declares that `Password` is strictly equivalent to `string`. Type aliases are purely structural, they don't create a more derived type in the internal type graph.

By contrast, **classes** are nominally typed and therefore introduce new semantic nodes:

```
Password is alias of string
Passphrase is string (
	
)

password is Password = "abc"
passphrase = Passphrase("this is my passphrase")

User (
	export password is var Password
	export passphrase is var Passphrase
)

user = User()

// ✅ works because Password is just an alias for string
user.password = passphrase 

// ❌ fails because even though it's based on string,
// Passphrase is a distinct, more derived type node
user.passphrase = password
```

To be clear, a bare alias does nothing more than rename the type. Its main use comes when building complex type shapes.

---

# Complex Type Aliases 

Most type aliases in the code will have more complexity than the previous example. Below is a walk through of the possible variants the type alias syntax.

Without parens:

```
MyAlias1 is alias of string
MyAlias2 is alias of string or boolean or int
```

More complex examples require surrounding parens. If a type alias uses the surrounding parens, the items within that parens group are divided into 3 sections:

1. Invariant declarations (optional)
2. Generic parameters (optional)
3. Type expression

```
MyAlias is alias of (
	// declares
	// generic parameters
	// type expression
)
```

### Declaring Generic Type Parameters

- `X is type` — declares a generic type parameter with no constraints.
- `X is type of Y` — constrains the generic to a specific type or primitive.

```first
Foo(T is type, U is type of Node) (
	
)
```

- `T is type` → a completely unconstrained generic.
- `U is type of number` → a generic constrained to `number`.

# Critical Differences With TypeScript

TypeScript, like most languages, use a different semantic region for generic type parameters and standard parameters. First collapses this distinction because managing two separate parameter spaces is too difficult to represent visually, and also makes the model more academic and less friendly to newcomers.

First merges the generic type parameters in with standard type parameters. This is done in an effort to streamline the display of the code, especially in the editor. It therefore needs to have slightly different semantics. For example, observe the differences between TypeScript and First.

TypeScript:

```first
class Foo<T extends Node>
{
	method<U>(a: T, b: U)
	{
	
	}
}
```

First:

```
Foo (
	(T is type of Node)
	
	method(U is type, a is T, b is U) (
	
	)
)
```

Key differences:

- Class generic type parameters go on the constructor, not the class definition itself
- Bare `<T>`-style definitions don't have a parallel. Instead, you need to explicitly annotate `T is type`
- Type constraints are defined by using the `is type of X` syntax instead of `extends`.

## Unions and Intersections

First supports the same unions and intersections features as TypeScript, however, instead of the `|` and `&` operators, First uses the `or` and `and` keywords.

```
Primitives is alias of string or int or boolean
ApiObject is alias of ApiObjectV1 and ApiObjectV2
```

Unlike TypeScript, intersections over primitives are prohibited.

---

## Type Extractions

Type Extractions allow you to **carry type information from values, functions, or class structures into type definitions**. They are a powerful way to reflect the structure of code into type aliases.

By default, extractions that include names and types produce **object shapes**. They can be coerced into arrays or singular types when the context requires.

### **Extraction Targets**

|Target|Extraction Result|
|---|---|
|`Function`|object with key = function name, value = return type|
|`Function.return`|return type of function (key = `null`)|
|`Function.arguments`|array of `[name: string, type: Type]` tuples for each argument|
|`Function.arguments[n]`|single `[name, type]` tuple; negative indices supported like JavaScript arrays|
|`Class.method`|same as `Function` for class methods|
|`Class.method.return`|return type of the method|
|`Class.method.arguments`|array of `[name, type]` tuples for method parameters|
|`Class.arguments`|array of `[name, type]` tuples for constructor parameters|

### **Rules and Behaviours**

1. **Default Shape**
    - All extractions default to **object shapes**.
    - Keys are names (argument names, method names, function names), and values are the associated types.
    - If the extraction is used in a **context expecting an array**, only the list of types is carried forward.
    - If the extraction is used in a **context expecting a single type**, a chain-of-types is produced.

2. **Arguments Tuples**
    - `.arguments` returns arrays of **two-element tuples**: `[name: string, type: Type]`.
    - Access `[n]` to retrieve a specific tuple; negative indices (`-1`, `-2`, …) are supported.

3. **Optional and Default Parameters**
    - Default values are **erased**; only the type matters.
    - Optional parameters are flattened into `T or null` to maintain array consistency.

4. **Class Methods**
    - Only real class methods can be extracted; fields storing function types cannot be introspected for arguments.
    - Extracted `.method` types operate entirely at the **type level**.

5. **Nested Extractions**
    - Extractions can be nested inside objects:


```first
Nested is alias of {
	f1 is Fn1.return
	f2 is Fn2.arguments
}
```

- `f2` resolves to an **object** where keys are argument names and values are argument types.
- Arrays or chain-of-types only appear if explicitly coerced.

---

### **Examples**

```first
// Extract return type of a function
Result is alias of someFn.return

// Extract all argument types as tuples
Args is alias of someFn.arguments
FirstArg is alias of someFn.arguments[0]
LastArg is alias of someFn.arguments[-1]

// Extract class method arguments
MethodArgs is alias of SomeClass.someMethod.arguments
MethodReturn is alias of SomeClass.someMethod.return

// Extract constructor arguments
CtorArgs is alias of SomeClass.arguments
```

```first
// extraction produces an object
TExample is alias of {
	innerResult is SomeClass.someMethod.return
	argsArray is [ ...SomeFn.arguments ] // coerces to array of types only
	argsObject is SomeFn.arguments
	argsObjectExtended is {
		...SomeFn.arguments
		property1 is string
		property2 is string
	}
}
```

---

This structure eliminates latent objects and makes **extractions predictable**:

- default → object shape
- coerced → array of types or single type chain

It also preserves argument names and types consistently for better type reflection.

---

## Control Flow Type Attestations

Because generics in First are monomorphic, you can **inspect type variables in control flow**:

```first
foo(T is type of Animal) (
	// Affirmative attestation
	if (T is Dog) (
		console.log("T is a dog")
	)
	
	// Negated attestation
	if (T is not Cat) (
		console.log("T is not a cat")
	)
)
```

- `if (T is X)` → checks if the generic type equals `X`
- `if (T is not X)` → checks if the generic type is not `X`

---

## Built-in Utilities

First includes standard type utilities. Parentheses are used instead of angle brackets:

```first
Partial(T)
Pick(T, K)
Omit(T, K)
Exclude(T, U)
Extract(T, U)
Capitalize(T)
Uncapitalize(T)
UpperCase(T)
LowerCase(T)
ToSnake(T)
FromSnake(T)
```

These can be used **anywhere a type is allowed**.


---

## Examples

### Simple Alias

```first
Password is nameof string
```


```first
TFoo is alias of Object1 and Object2

ObjectA is alias of (
	T is type
	{ a is T }
)

ObjectB is alias of (
	T is type
	{ b is T }
)

MyAlias3 is alias of (
	declare someInvariant
	
	T1 is type
	T2 is type of TFoo
	
	ObjectA(T1) and ObjectB(T2)
)
```

### Generic Function with Type Attestation

```first
wrapInArray(T is type, arg is T or T[]) (
	if (arg is T[]) (
		return arg
	)
	
	return [arg]
)
```

---

## Provisional Inline Type Rules

The following language decisions were made while introducing the initial parser grammar. Preserve them when this document is reorganized and reconciled with the alias and attestation documentation.

First deliberately distinguishes between type expressions written inline in annotations and the fuller type language available to `alias of` declarations. Inline annotations must remain visually linear. When a type requires precedence grouping or mixes composition operators, it must be given a name with `alias of`.

Inline type positions currently include function parameters, generic constraints, function return annotations, fields, and local bindings. They accept:

- primitive, named, and generic-parameter types;
- generic application using parentheses, such as `Result(string, User)`;
- any number of nested array suffixes, such as `T[][]`;
- flat unions of any length using `or`;
- flat intersections of any length using `and`;
- the `editable` type qualifier.

Standalone grouping parentheses are not permitted in inline type expressions. Mixing `and` and `or` in one inline expression is also not permitted. These forms must be moved into an alias:

```first
// Not legal inline:
load() is (Cached or Remote)[] (...)
combine() is A and B or C (...)

// Name the composition instead:
LoadSource is alias of Cached or Remote
Combined is alias of A and B or C

load() is LoadSource[] (...)
combine() is Combined (...)
```

Parentheses used for generic application are not grouping parentheses and remain legal inline:

```first
load() is Result(Error, User[]) (...)
```

Generic type parameters share the ordinary parameter list with runtime parameters. They use `T is type` or `T is type of Constraint`; constraints accept the same inline type-expression space as other annotations. Canonically, generic parameters appear before runtime parameters. Parsing may retain a noncanonical source order so malformed or merge-affected code can still enter the editor; validation and canonical reordering are separate concerns.

Generic application never uses angle brackets. Angle brackets remain available for markup and comparison syntax.

`var` and `editable` represent different concerns:

- `var` belongs to a binding and permits that binding to be rebound. It is not part of a type and cannot appear in a type alias.
- `editable` qualifies the referenced value/type as editable. It may therefore appear in an inline annotation or a type alias.

Whether parameters may use `var` remains an independent language decision. It does not affect the type-expression grammar.

TypeScript-style literal string and number types are not part of First. Their common role as ad hoc enumerations is served by First's explicit selection types, which should be used instead.

Primitive types may participate in unions:

```first
PrimitiveResult is alias of string or int or null
```

Primitive types may not participate in intersections. Expressions such as `int and string` describe an impossible value rather than a useful First type. Intersection operands must therefore be named or composed non-primitive types; this restriction is represented directly by the intersection production rather than deferred to type checking.
