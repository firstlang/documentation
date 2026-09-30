# Operator Overloading

Only primitive classes can overload operators.

The overloadable binary operators are:

- `+`
- `-`
- `*`
- `/`
- `\`
- `%`
- `**`
- `==`
- `<`
- `<=`
- `>`
- `>=`

`!=` derives from `==`. `!==` derives from `===`.

Compound assignment uses the underlying binary operator. For example, `+=` uses `+`.

Strict equality (`===`), bitwise operators, logical operators, coalescing operators, ternaries, assignment, `key of`, and postfix `throw` are not overloadable.
