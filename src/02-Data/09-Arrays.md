
	
Arrays can be defined as either fixed-length, or as variable length.
A variable-length array works just like the array in JavaScript / TypeScript:

```C#
🥚 variableLength is var number[]
```

A fixed-length array is created by specifying the length of the array in the type definition.
Note that for this syntax to work, the "type area" of the language would have to be part of
the program flow, not a pre-processor only thing like how TypeScript works.

```first
// Initialize a 10 number array
// Fixed-length array initializers don't need an assignment.
// They're initialized with the default value of the type (0, empty string, false, etc)
🥚 fixedArray is number[10]
🥚 someLength = 10
🥚 fixedArray is number[someLength]
```

Array contents are immutable by default. If you want to make the contents mutable, you have to use the `editable` keyword.

```first
items is editable number[]

0 to 10 each i (
	items.push(i)
)
```

To make the array both have a mutable binding and mutable contents, you'd need to use both annotations together (and hope to not get fired)

```first
items is var editable number[]

0 to 10 each i (
	items.push(i)
)
```

To make an array with a mutable binding, editable contents, but a fixed length, the code would look like:

```first
items is var editable number[10]

0 to 10 each i (
	items.push(i)
)
```

Arrays are auto-initialized as such. They don't need to have equal signs with empty vars put after them.

```first
// These are equivalent
items is number[]
items is number[] = [1, 2]
```
