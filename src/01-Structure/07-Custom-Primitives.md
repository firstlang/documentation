Custom primitives define nominal value types.

They are used for branded primitives and small value-shaped aggregates. Custom primitives live in a separate type universe from reference classes. A custom primitive cannot inherit from a reference class, and a reference class cannot inherit from a custom primitive.

```first
UserId is u64
```

Custom primitives are immutable values. They do not have object identity, are not reference counted, and do not support ghosts.

## Bases

A root custom primitive may use one of three base forms:

- `primitive`
- a concrete primitive, such as `u64`, `string`, `f32`, or `boolean`

Custom primitives also support single inheritance from another custom primitive. They do not support multiple inheritance. A custom primitive chooses one base.


## Concrete Primitive Bases

Inheriting from a concrete primitive creates a branded primitive with the same underlying representation.

```first
UserId is u64 (
)

id = UserId(123)
```

`UserId` is a distinct nominal type, but its stored value is a `u64`.

Custom primitives based on a concrete primitive receive an implicit first construction parameter. The parameter is compatible with the base primitive. The editor may show this parameter in the function signature, but it does not appear in the underlying text file.

```first
BracketedString is string (
)

value = BracketedString("hello")
```

Custom primitive chains rooted in a concrete primitive cannot declare stored fields. Once the chain has chosen a concrete primitive representation, its shape is fixed.

```first
UserId is u64 (
	// > ❌ compiler notice
	label is string
)
```

## Fresh Primitive Shapes

Inheriting directly from `primitive` creates a fresh primitive shape.

```first
Point is primitive (
	x is f32
	y is f32
)
```

Fields are allowed only for custom primitive chains rooted at `primitive`.

Aggregate custom primitives may contain primitive fields and reference fields.

```first
LabeledPoint is primitive (
	x is f32
	y is f32
	label is string
	owner is User
)
```

The custom primitive itself is still a value. If it contains a reference field, that field is a reference value stored inside the primitive value.

Custom primitives based directly on `primitive` do not receive an implicit first construction parameter, because there is no single base representation to pass in.

## Custom Primitive Inheritance

A custom primitive may inherit from one other custom primitive.

```first
Vector2 is primitive (
	x is f32
	y is f32
)

Vector3 is Vector2 (
	z is f32
)
```

The root of the custom primitive chain determines its representation family.

If the chain is rooted at `primitive`, descendants may add stored fields.

```first
Vector4 is Vector3 (
	w is f32
)
```

If the chain is rooted at a concrete primitive, descendants keep the same scalar representation family and cannot add stored fields.

```first
UserId is u64 (
)

AdminUserId is UserId (
)
```

Custom primitive inheritance creates nominal compatibility and an ancestor view. A value accessed through an ancestor type shows the ancestor's member and field surface.

```first
v is Vector2 = Vector3(1, 2, 3)

v.x
v.y
// > ❌ compiler notice
v.z
```

The descendant view can be restored with an attestation. If the attestation fails, the access path short-circuits.

```first
zValue = (v is Vector3).z
// > f32 or null if the attestation fails
console.log(zValue)
```

## Constructors

Custom primitives may define constructors.

For concrete primitive bases, construction begins with the implicit first parameter. Inside the constructor, `this` refers to the current custom primitive value.

A constructor may call `super(value)` to replace the underlying base value for the value being constructed.

```first
BracketedString is string (
	constructor() (
		super("[" + this + "]")
	)
)

value = BracketedString("hi")
// > "[hi]"
console.log(value)
```

The value passed to `super(...)` must be compatible with the base primitive. If the constructor does not call `super(...)`, the original implicit value is used.

For custom primitives based directly on `primitive`, there is no base value to replace, so `super(...)` is not available.

```first
Point is primitive (
	x is f32
	y is f32
	
	constructor(x is f32, y is f32) (
		this.x = x
		this.y = y
	)
)
```

Custom primitive constructors follow the ordinary constructor result rules. They may fall through to the constructed primitive value, return a compatible custom primitive value, or bomb. A bombing call evaluates to `bomb`, so the construction expression has type `CustomPrimitive or bomb`.

An empty custom-primitive body may be omitted. When a body is present, it keeps the ordinary custom-primitive member grammar. It may also contain anchors owned by the custom primitive declaration.

## Members

Custom primitives may define functions and getters. Scalar-backed custom primitives may also define operator overloads. They may also use nested spaces to group otherwise permitted members. A group does not relax the custom primitive's representation rules: scalar-backed custom primitive chains cannot add fields at any depth, while fresh primitive shapes treat grouped fields as part of the same primitive storage. Directly accessible program-owned members belong in a same-name space.

```first
Counter is u64 (
	next() is Counter (
		return Counter(this + 1)
	)
)
```

Custom primitives cannot define setters or ghosts.

Functions on custom primitives do not mutate the existing value. They return new values when a changed value is needed.

## Shadowing

A custom primitive can define a member with the same name as a member available on its base. This shadows the base member for the custom primitive.

There is no virtual dispatch. Member lookup is selected by the static type of the value.

When a shadowing member needs to call the base behavior, it can use `super`.

```first
MyString is string (
	toString() is string (
		return "MyString(" + super.toString() + ")"
	)
)
```

For ordinary operations, use `this`.

```first
Score is int (
	add(amount is int) is Score (
		return Score(this + amount)
	)
)
```

## Equality

First has strict equality and loose equality.

Strict equality requires the custom primitive and the underlying value to match.

Loose equality compares the underlying data. Branded primitive types may be different.

```first
UserId is u64 (
)

OrderId is u64 (
)

user = UserId(10)
order = OrderId(10)

// > false
user === order
// > true
user == order
```

For numeric custom primitives with different concrete numeric representations, loose equality follows the normal numeric conversion rules.

## Units

Units are compile-time numeric tags, not custom primitives.

```first
cm is unit

width = 10cm
```

Unit syntax and unit operator behavior are described in [[04-Units]].

## Operator Overloads

Operator overloads are supported only for scalar-backed custom primitives.

They may be defined inside a scalar-backed custom primitive.

```first
Score is i32 (
	
	operator + (this, that is Score) (
		return Score(this + that)
	)
)
```

Operator overloads use the `operator` keyword. The canonical operands are `this` and `that`. `this` is untyped and means the containing custom primitive; `that` is typed with ordinary `is` syntax.

```first
UserId is u64 (
	operator == != (this, that is UserId) (
		return this == that
	)
)
```

Extensions cannot define operator overloads.
