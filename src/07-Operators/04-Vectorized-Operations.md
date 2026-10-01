# Vectorized Operations

Vectorized operations apply an ordinary scalar operator across array elements.

```first
values = [1, 2, 3]
scaled = values * 2 // [2, 4, 6]
```

Vectorized operations are element-wise. They are not matrix operations, dot products, reductions, range operations, or object-field operations.

```first
range = 1 to 3
range * 2 // Notice

range.toArray() * 2 // [2, 4, 6]
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

scores + 5 // [15, 25, 35]
scores >= 20 // [false, true, true]
scores & 1 // Notice
```

# Scalar Broadcasting

When one operand is an array and the other is not, the non-array operand is applied to every element. Operand order is preserved.

```first
values = [2, 4, 8]

values - 1 // [1, 3, 7]
10 - values // [8, 6, 2]

values / 2 // [1, 2, 4]
16 / values // [8, 4, 2]
```

# Array Pairing

When both operands are arrays, elements are paired by index.

```first
[1, 2, 3] + [10, 20, 30] // [11, 22, 33]
```

If one array is shorter, First pads the shorter side with the operator's identity value.

```first
[1, 2, 3] + [10] // [11, 2, 3]
[1, 2, 3] * [10] // [10, 2, 3]
```

The additive identity is `0`. The multiplicative identity is `1`. Operators without a usable identity for the element operation use that operation's zero-value fallback for missing lanes.

Empty array operations produce empty arrays.

```first
[] * 10 // []
[] + [] // []
```

# Result Types

The result element type is the ordinary scalar result type of the element operation.

```first
left is i32[] = [1, 2]
right is i64[] = [3, 4]

total = left + right // i64[]
```

Unit adoption, related-unit adoption, primitive-class operator overloads, and numeric promotion work the same way they do for scalar operators.

```first
distances = [1m, 2m, 3m]
distances + 2 // [3m, 4m, 5m]

left is u8[] = [1, 2]
right is u64[] = [10, 20]
sum = left + right // u64[]
```

If every array operand has the same statically known fixed length, the result is fixed-length. Otherwise the result is variable-length.

```first
left is int[3] = [1, 2, 3]
right is int[3] = [10, 20, 30]
fixed = left + right // int[3]

values is int[] = getValues()
scaled = values * 2 // int[]
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

customers * animals // Notice
```

# Comparisons

Vectorized ordering comparisons return `boolean[]`.

```first
scores = [10, 20, 30]

scores > 15 // [false, true, true]
scores <= [10, 10, 40] // [true, false, true]
```

Bare array equality is whole-array equality, not a vectorized boolean-array operation. It compares array length and then compares corresponding elements using the relevant scalar equality behavior for each position.

```first
[1, 2] == [1, 2] // true
[1, 2] == [1, 3] // false
```

# Compound Assignment

Compound assignment uses the vectorized binary operation and assigns the result back to the binding. It does not mutate array contents in place as a language-level behavior.

```first
var items = [1, 2, 3]
items += 1 // items becomes [2, 3, 4]

fixed = [1, 2, 3]
fixed += 1 // Notice
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
["a", "b", "c"] + ["x", "y", "z"] // ["ax", "by", "cz"]
["a", "b"] * 2 // Notice
```

No other string operator becomes valid because of vectorization.

# Invalid Operations

Invalid vectorized operations produce a notice at the outer operator. The diagnostic should include the failing element operation when that is useful.

```first
names = ["Ada", "Grace"]
names - 1 // Notice
```

Object values do not vectorize by field. Types that need field-wise arithmetic should define an explicit operator.

```first
point = { x = 1, y = 2 }
point * 2 // Notice

Vector2 is primitive (
	x is f64
	y is f64
	
	* (scale is f64) is Vector2 (
		return Vector2(this.x * scale, this.y * scale)
	)
)
```
