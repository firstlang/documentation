Primitive classes define nominal value types.

They are used for branded primitives, units, and small value-shaped aggregates. Primitive classes live in a separate type universe from reference classes. A primitive class cannot inherit from a reference class, and a reference class cannot inherit from a primitive class.

```first
UserId is u64
```

Primitive classes are immutable values. They do not have object identity, are not reference counted, and do not support ghosts.

## Bases

A root primitive class may use one of three base forms:

- `primitive`
- a concrete primitive, such as `u64`, `string`, `f32`, or `boolean`
- a primitive group, such as `AnyInteger`, `AnySignedInteger`, or `AnyNumeric`

Primitive classes also support single inheritance from another primitive class. They do not support multiple inheritance. A primitive class chooses one base.

The provisional `numeric` annotation is not a primitive class base. To define a primitive class over a family of number types, use a primitive group instead.

## Concrete Primitive Bases

Inheriting from a concrete primitive creates a branded primitive with the same underlying representation.

```first
UserId is u64 (
)

id = UserId(123)
```

`UserId` is a distinct nominal type, but its stored value is a `u64`.

Primitive classes based on a concrete primitive receive an implicit first construction parameter. The parameter is compatible with the base primitive. The editor may show this parameter in the function signature, but it does not appear in the underlying text file.

```first
BracketedString is string (
)

value = BracketedString("hello")
```

Primitive class chains rooted in a concrete primitive cannot declare stored fields. Once the chain has chosen a concrete primitive representation, its shape is fixed.

```first
UserId is u64 (
	label is string // ❌ compiler notice
)
```

## Primitive Group Bases

Inheriting from a primitive group creates a parametric primitive class over the concrete primitives in that group.

```first
DatabaseId is AnyUnsignedInteger (
)

id = DatabaseId(123)
```

`AnyUnsignedInteger` is not ordinary inheritance. It is a constrained provisional base. The concrete representation follows the same provisional number rules used elsewhere in the language, but it is constrained to the primitive types in the group.

Primitive group bases also receive an implicit first construction parameter. The parameter is compatible with the group.

Primitive class chains rooted in primitive groups cannot declare stored fields.

## Fresh Primitive Shapes

Inheriting directly from `primitive` creates a fresh primitive shape.

```first
Point is primitive (
	x is f32
	y is f32
)
```

Fields are allowed only for primitive class chains rooted at `primitive`.

Aggregate primitive classes may contain primitive fields and reference fields.

```first
LabeledPoint is primitive (
	x is f32
	y is f32
	label is string
	owner is User
)
```

The primitive class itself is still a value. If it contains a reference field, that field is a reference value stored inside the primitive value.

Primitive classes based directly on `primitive` do not receive an implicit first construction parameter, because there is no single base representation to pass in.

## Primitive Class Inheritance

A primitive class may inherit from one other primitive class.

```first
Vector2 is primitive (
	x is f32
	y is f32
)

Vector3 is Vector2 (
	z is f32
)
```

The root of the primitive class chain determines its representation family.

If the chain is rooted at `primitive`, descendants may add stored fields.

```first
Vector4 is Vector3 (
	w is f32
)
```

If the chain is rooted at a concrete primitive or primitive group, descendants keep the same scalar representation family and cannot add stored fields.

```first
UserId is u64 (
)

AdminUserId is UserId (
)
```

Primitive class inheritance creates nominal compatibility and an ancestor view. A value accessed through an ancestor type shows the ancestor's member and field surface.

```first
v is Vector2 = Vector3(1, 2, 3)

v.x
v.y
v.z // ❌ compiler notice
```

The descendant view can be restored with an attestation. If the attestation fails, the access path short-circuits.

```first
zValue = (v is Vector3).z
console.log(zValue) // f32 or null if the attestation fails
```

## Constructors

Primitive classes may define constructors.

For concrete primitive and primitive group bases, construction begins with the implicit first parameter. Inside the constructor, `this` refers to the current primitive class value.

A constructor may call `super(value)` to replace the underlying base value for the value being constructed.

```first
BracketedString is string (
	constructor() (
		super("[" + this + "]")
	)
)

value = BracketedString("hi")
console.log(value) // "[hi]"
```

The value passed to `super(...)` must be compatible with the base primitive or primitive group. If the constructor does not call `super(...)`, the original implicit value is used.

For primitive classes based directly on `primitive`, there is no base value to replace, so `super(...)` is not available.

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

Primitive constructors follow the ordinary constructor result rules. They may fall through to the constructed primitive value, return a compatible primitive-class value, or bomb. A bombing call evaluates to `bomb`, so the construction expression has type `PrimitiveClass or bomb`.

An empty primitive-class body may be omitted. When a body is present, it keeps the ordinary primitive-class member grammar. It may also contain anchors owned by the primitive class declaration.

## Members

Primitive classes may define functions and getters. Scalar-backed primitive classes may also define operator overloads. They may also use nested spaces to group otherwise permitted members. A group does not relax the primitive class's representation rules: scalar-backed primitive chains cannot add fields at any depth, while fresh primitive shapes treat grouped fields as part of the same primitive storage. Directly accessible program-owned members belong in a same-name space.

```first
Counter is u64 (
	next() is Counter (
		return Counter(this + 1)
	)
)
```

Primitive classes cannot define setters or ghosts.

Functions on primitive classes do not mutate the existing value. They return new values when a changed value is needed.

## Shadowing

A primitive class can define a member with the same name as a member available on its base. This shadows the base member for the primitive class.

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

Strict equality requires the primitive class and the underlying value to match.

Loose equality compares the underlying data. Branded primitive types may be different.

```first
UserId is u64 (
)

OrderId is u64 (
)

user = UserId(10)
order = OrderId(10)

user === order // false
user == order // true
```

For numeric primitive classes with different concrete numeric representations, loose equality follows the normal numeric conversion rules.

## Units

A numeric-backed primitive class marked with `declare unit` can be used as a unit suffix.

```first
cm is AnyNumeric (
	declare unit
)

width = 10cm
```

Unit syntax and unit operator behavior are described in [[04-Units]].

## Operator Overloads

Operator overloads are supported only for scalar-backed primitive classes.

They may be defined inside a scalar-backed primitive class.

```first
Score is i32 (
	
	operator + (this, that is Score) (
		return Score(this + that)
	)
)
```

Operator overloads use the `operator` keyword. The canonical operands are `this` and `that`. `this` is untyped and means the containing primitive class; `that` is typed with ordinary `is` syntax.

```first
UserId is u64 (
	operator == != (this, that is UserId) (
		return this == that
	)
)
```

Extension classes cannot define operator overloads.
