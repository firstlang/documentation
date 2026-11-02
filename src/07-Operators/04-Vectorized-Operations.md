# Vectorized Operations

Vectorized operations apply an ordinary scalar operator across array elements.

```first
values = [1, 2, 3]
// > [2, 4, 6]
scaled = values * 2
```

Vectorized operations are element-wise. They are not matrix operations, dot products, reductions, range operations, or object-field operations.

```first
range = 1 to 3
// > Notice
range * 2

// > [2, 4, 6]
range.toArray() * 2
```

# Supported Operators

The vectorized arithmetic and ordering operators are:

- `+`
- `-`
- `*`
- `/`
- `\`
- `%`
- `**`
- `<`
- `<=`
- `>`
- `>=`

Strict equality, bitwise operators, logical operators, coalescing operators, ternaries, assignment, `key of`, and postfix `throw` do not vectorize.

```first
scores = [10, 20, 30]

// > [15, 25, 35]
scores + 5
// > [false, true, true]
scores >= 20
// > Notice
scores & 1
```

# Scalar Broadcasting

When one operand is an array and the other is not, the non-array operand is applied to every element. Operand order is preserved.

```first
values = [2, 4, 8]

// > [1, 3, 7]
values - 1
// > [8, 6, 2]
10 - values

// > [1, 2, 4]
values / 2
// > [8, 4, 2]
16 / values
```

# Array Pairing

When both operands are arrays, elements are paired by index.

```first
// > [11, 22, 33]
[1, 2, 3] + [10, 20, 30]
```

If one array is shorter, First pads the shorter side with the operator's identity value.

```first
// > [11, 2, 3]
[1, 2, 3] + [10]
// > [10, 2, 3]
[1, 2, 3] * [10]
```

The additive identity is `0`. The multiplicative identity is `1`. Operators without a usable identity for the element operation use that operation's zero-value fallback for missing lanes.

Empty array operations produce empty arrays.

```first
// > []
[] * 10
// > []
[] + []
```

# Result Types

The result element type is the ordinary scalar result type of the element operation.

```first
left is i32[] = [1, 2]
right is i64[] = [3, 4]

// > i64[]
total = left + right
```

Unit adoption, unit operator overloads, custom-primitive operator overloads, and numeric promotion work the same way they do for scalar operators.

```first
distances = [1m, 2m, 3m]
// > [3m, 4m, 5m]
distances + 2

left is u8[] = [1, 2]
right is u64[] = [10, 20]
// > u64[]
sum = left + right
```

If every array operand has the same statically known fixed length, the result is fixed-length. Otherwise the result is variable-length.

```first
left is int[3] = [1, 2, 3]
right is int[3] = [10, 20, 30]
// > int[3]
fixed = left + right

values is int[] = getValues()
// > int[]
scaled = values * 2
```

# Mixed Element Selections

When an array's element type is a selection, First infers the result element type from the statically valid element operations. At runtime, valid lanes produce the scalar result. Invalid lanes produce the zero value of the inferred result element type.

```first
items is (int or Customer)[] = [1, Customer, 3]

scaled = items * 10
// int[] containing [10, 0, 30]
```

The same rule applies when both sides contain selections.

```first
left is (int or Customer)[] = [1, Customer]
right is (f64 or Animal)[] = [2.5, Animal]

merged = left * right
// f64[] containing [2.5, 0.0]
```

If no element pairing has a valid scalar operation, the operation produces an empty impossible-value array and the compiler reports a notice.

```first
customers is Customer[] = getCustomers()
animals is Animal[] = getAnimals()

// > Notice
customers * animals
```

# Comparisons

Vectorized ordering comparisons return `boolean[]`.

```first
scores = [10, 20, 30]

// > [false, true, true]
scores > 15
// > [true, false, true]
scores <= [10, 10, 40]
```

Bare array equality is whole-array equality, not a vectorized boolean-array operation. It compares array length and then compares corresponding elements using the relevant scalar equality behavior for each position.

```first
// > true
[1, 2] == [1, 2]
// > false
[1, 2] == [1, 3]
```

# Compound Assignment

Compound assignment uses the vectorized binary operation and assigns the result back to the binding. It does not mutate array contents in place as a language-level behavior.

```first
var items = [1, 2, 3]
// > items becomes [2, 3, 4]
items += 1

fixed = [1, 2, 3]
// > Notice
fixed += 1
```

Implementations may reuse storage only when ordinary ownership analysis proves the reuse is unobservable. Debug and production builds have the same language semantics.

# Evaluation

The left operand expression evaluates once, then the right operand expression evaluates once. Vectorized result elements are observed in ascending index order.

```first
result = getLeftArray() + getRightArray()
// getLeftArray() runs once before getRightArray().
```

Implementations may optimize or parallelize only when doing so cannot change observable behavior, diagnostics, cleanup, or result values.

# Strings

String arrays support `+` because scalar string `+` is valid. This is element-wise concatenation.

```first
// > ["ax", "by", "cz"]
["a", "b", "c"] + ["x", "y", "z"]
// > Notice
["a", "b"] * 2
```

No other string operator becomes valid because of vectorization.

# Invalid Operations

Invalid vectorized operations produce a notice at the outer operator. The diagnostic should include the failing element operation when that is useful.

```first
names = ["Ada", "Grace"]
// > Notice
names - 1
```

Object values do not vectorize by field. Types that need field-wise arithmetic should define an explicit operator.

```first
point = { x = 1, y = 2 }
// > Notice
point * 2

Vector2 is primitive (
	x is f64
	y is f64
	
	* (scale is f64) is Vector2 (
		return Vector2(this.x * scale, this.y * scale)
	)
)
```
