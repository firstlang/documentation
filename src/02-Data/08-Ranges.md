# Ranges

Ranges in First are immutable copy values that represent a regularly-spaced sequence between two numeric boundary values.

They are defined with the following syntax:

```first
<initial-value> <to | til> <max-value> ?(step <step-value>)
```

The `to` keyword creates an inclusive range. The end value may appear during iteration if it lands exactly on a step.

The `til` keyword creates an exclusive range. The end value never appears during iteration, even if it lands exactly on a step.

The optional `step` keyword controls the granularity of the range. If `step` is omitted, it is assumed to be `1`.

```first
0 to 5 step 2 // Iterates 0, 2, 4
0 to 6 step 2 // Iterates 0, 2, 4, 6
0 til 6 step 2 // Iterates 0, 2, 4
```

# Type Rules

`<initial-value>`, `<max-value>`, and `<step-value>` must be primitive number values, with the following exceptions:

- Character values are allowed. A character range yields `char` values.
- Floating point values (`f16`, `f32`, `f64`, `f128`) are not valid range values. If a float appears in a range, a saw appears and the value is converted to the equivalently-sized integer type, truncating any fractional component.

The types of `<initial-value>`, `<max-value>`, and `<step-value>` may be different, as long as they conform to the range rules.

The range type is the narrowest type that can represent the start, end, and step values after any required conversion. Provisional numbers are pinned to that shared range type.

```first
0 to 10 step 0.25 // Range(i64.2)
```

# Empty Ranges

A range is empty when its step can never move from the start value toward the end value, or when the start and end values leave no values to iterate.

```first
0 to 0 // Empty range
0 til 0 // Empty range
0 to 10 step -1 // Empty range
10 to 0 step 1 // Empty range
```

If an empty range can be determined statically, the compiler emits a notice. If it can only be discovered at runtime, the range is simply empty.

```first
lo = getLowValue()
hi = getHighValue()
step = getStep()

range = lo to hi step step // May be empty at runtime
```

`step 0` is not allowed. If the step value is statically known to be `0`, the compiler emits a notice. If a runtime value evaluates to `0`, creating the range is a runtime error.

# Reverse Ranges

Ranges may move forward or backward. A backward range must have a negative step.

```first
5 to 0 step -2 // Iterates 5, 3, 1
6 to 0 step -2 // Iterates 6, 4, 2, 0
6 til 0 step -2 // Iterates 6, 4, 2
```

Once a range is created, its start, end, step, and inclusivity cannot be changed.

# Range Iteration

Ranges support iteration with the `each` keyword:

```first
0 to 10 each i (
	console.log(i)
)
```

The iteration type is the range type.

# Range Shape

Ranges have the following definition:

```first
📦 Range(T is numeric) (
	declare copy
	
	start is T
	end is T
	step is T
	inclusive is boolean
	
	get length is T
	
	// Returns whether the specified value is between start
	// and end. If useStep is true, the value must also land
	// on one of the range's steps.
	contains(value is T, useStep is boolean = false) is boolean
	
	// Returns whether there is overlapping area between this
	// range and another range. Step alignment is ignored.
	overlaps(other is Range(T)) is boolean
	
	reverse() is Range(T)
	
	// Produces the range intersection, or an empty range if
	// the ranges contain no overlapping area. Step alignment
	// is ignored.
	intersect(other is Range(T)) is Range(T)
	
	// Produces the smallest range that contains both ranges,
	// filling in any intermediate hole. Step alignment is ignored.
	union(other is Range(T)) is Range(T)
	
	// Produces zero, one, or two ranges where the area of the
	// second range has been removed from this range. Step
	// alignment is ignored.
	difference(other is Range(T)) is Range(T)[]
	
	isSubsetOf(other is Range(T)) is boolean
	
	isSupersetOf(other is Range(T)) is boolean
	
	toArray() is T[]
)
```

There is no range constructor. Ranges must be created with the range literal syntax.

# Equality

Ranges are copy values and can be equality-tested with `==` and `===`.

Loose equality with `==` compares the iterated sequence. Compatible numeric types may compare equal without forcing the author to cast.

```first
range1 = 0 to 6 step 2
range2 = 0 til 7 step 2

console.log(range1 == range2) // true
```

Strict equality with `===` compares the exact range shape, including `start`, `end`, `step`, `inclusive`, and type.

```first
range1 = 0 to 6 step 2
range2 = 0 til 7 step 2

console.log(range1 === range2) // false
```

# Binary Operations

Ranges do not support scalar binary operations.

```first
range = 0 to 5

range + 1 // Notice
range * 10 // Notice
```

When a transformed range is needed, create a new range explicitly from the original range values.

# Saws

Ranges follow the same rules around number type conversion as described in [[04-Provisional-Numbers]]. Ranges that are typed more narrowly than their assignment target are automatically up-cast. If the conversion is potentially lossy, a saw appears.

# Variable Bindings

Simple variable bindings may be used in a range, provided the values are numeric.

```first
lo = 10
high = 20
range = lo to high
```
