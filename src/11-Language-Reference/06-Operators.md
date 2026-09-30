# Operators

This page is the formal operator reference. For examples and ordinary usage guidance, see [[01-Operator-Usage]].

## Precedence

Higher rows bind more tightly.

| Precedence | Operators | Associativity | Notes |
| ---------- | --------- | ------------- | ----- |
| 16 | `.` `()` `[]` | Left | Member access, calls, indexing |
| 15 | postfix `throw` | Left | Bomb propagation |
| 14 | `not` `!` unary `+` unary `-` | Right | Prefix unary operators |
| 13 | `**` | Right | Exponentiation |
| 12 | `*` `/` `\` `%` | Left | Multiplication, fractional division, integer division, remainder |
| 11 | `+` `-` | Left | Addition, subtraction |
| 10 | `<<` `>>` `>>>` | Left | Shifts |
| 9 | `<` `<=` `>` `>=` | Left | Ordering comparisons |
| 8 | `==` `!=` `===` `!==` | Left | Equality comparisons |
| 7 | `&` | Left | Bitwise and |
| 6 | `^` | Left | Bitwise xor |
| 5 | `|` | Left | Bitwise or |
| 4 | `and` | Left | Boolean and; also participates in coalescing forms |
| 3 | `or` `??` `else` `catch` | Left | Boolean or and fallback operators |
| 2 | `? :` | Right | Ternary conditional |
| 1 | `=` `+=` `-=` `*=` `/=` `\=` `%=` `**=` | Right | Assignment recovery only |

Parentheses override precedence.

Assignment and compound assignment are statement-level forms. They do not produce a usable value, and chained assignment is invalid.

## Arithmetic

| Operator | Name | Result | Overloadable |
| -------- | ---- | ------ | ------------ |
| `+` | Addition | Numeric addition or supported value addition | Yes |
| `-` | Subtraction | Numeric subtraction or supported value subtraction | Yes |
| `*` | Multiplication | Numeric multiplication or supported value multiplication | Yes |
| `/` | Fractional division | Fractional quotient | Yes |
| `\` | Integer division | Integral quotient | Yes |
| `%` | Remainder | Remainder after division | Yes |
| `**` | Exponentiation | Left operand raised to right operand | Yes |

## Assignment

| Operator | Name | Notes |
| -------- | ---- | ----- |
| `=` | Assignment | Replaces the target value |
| `+=` | Add and assign | Uses `+` behavior |
| `-=` | Subtract and assign | Uses `-` behavior |
| `*=` | Multiply and assign | Uses `*` behavior |
| `/=` | Fractional-divide and assign | Uses `/` behavior |
| `\=` | Integer-divide and assign | Uses `\` behavior |
| `%=` | Remainder and assign | Uses `%` behavior |
| `**=` | Exponentiate and assign | Uses `**` behavior |

Compound assignment operators are not separately overloaded. They use the corresponding binary operator behavior.

## Comparison

| Operator | Name | Result | Overloadable |
| -------- | ---- | ------ | ------------ |
| `==` | Loose equal | `boolean` | Yes |
| `!=` | Not loose equal | `boolean` | No |
| `===` | Strict equal | `boolean` | No |
| `!==` | Not strict equal | `boolean` | No |
| `<` | Less than | `boolean` | Yes |
| `<=` | Less than or equal | `boolean` | Yes |
| `>` | Greater than | `boolean` | Yes |
| `>=` | Greater than or equal | `boolean` | Yes |

Loose equality is semantic equality. It may use operator overloads, numeric conversion, or unit underlying-value comparison where those rules apply. Strict equality is exact equality: the type identity and stored value must match, and the operator cannot be overloaded.

## Logical And Coalescing

| Operator | Name | Result |
| -------- | ---- | ------ |
| `!` | Boolean negation | `boolean` |
| `and` | Boolean and / true coalescing | Depends on context |
| `or` | Boolean or / false coalescing | Depends on context |
| `??` | Null coalescing | Left value without `null`, or fallback |
| `else` | False-or-null coalescing | Left value without `false` or `null`, or fallback |
| `catch` | Bomb coalescing | Left value without bomb, or fallback |

First does not use broad JavaScript-style truthiness. Coalescing operators fall through only for the condition named by the operator.

`&&` and `||` parse as legacy spellings for `and` and `or`. Tooling should rewrite them to the First spellings.

## Bitwise

| Operator | Name | Result |
| -------- | ---- | ------ |
| `&` | Bitwise and | Integer |
| `|` | Bitwise or | Integer |
| `^` | Bitwise xor | Integer |
| `<<` | Left shift | Integer |
| `>>` | Signed right shift | Integer |
| `>>>` | Unsigned right shift | Integer |

## Conditional And Name Operators

| Operator | Name | Result |
| -------- | ---- | ------ |
| `? :` | Ternary conditional | Selected branch type |
| `key of` | Name retrieval | `string` |
| postfix `throw` | Bomb propagation | Left value without bomb, or propagated bomb |

## Omitted Operators

First does not include `++` or `--`. Use explicit assignment or compound assignment instead.

## Evaluation And Recovery

Operands are evaluated left-to-right and exactly once. Short-circuiting forms evaluate the right operand only when their condition requires it. Compound assignment evaluates the target reference once, then the right operand once, then applies the underlying operator and writes back.

Invalid operator use produces a compiler notice. Implementations may recover by pruning, substituting, or simplifying invalid parts of the expression so the surrounding program can continue to parse and be analyzed. Recovery is implementation-defined and authors must not rely on the recovered value or execution result.
