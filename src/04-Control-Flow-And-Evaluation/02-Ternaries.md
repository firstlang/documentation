# Ternaries

Ternaries use the same syntax and basic behavior as TypeScript.

```first
label = enabled ? "Enabled" : "Disabled"
```

The format is:

```first
<condition> ? <true-expression> : <false-expression>
```

A ternary is an expression. It produces the value of the true arm when the condition passes, and the value of the false arm otherwise.

Use ternaries for simple two-arm expression selection. Use `if` for statement control flow, and use `matches` for multi-arm expression selection.

## Condition Types

A ternary condition follows the same rules as an `if` condition. It must be either:

- a `boolean`
- a nullable value
- an expression that produces one of those types, such as a comparison or an `is` check

Ternaries do not use TypeScript or JavaScript truthiness. Strings, numbers, arrays, objects, and other non-boolean values are not accepted as conditions merely because they could be interpreted as true or false.

```first
label = count ? "Non-zero" : "Zero"
// Invalid: numbers are not coerced to boolean

label = count !== 0 ? "Non-zero" : "Zero"
// Valid
```

The same rule applies to strings and collections. Use explicit checks.

```first
label = name !== "" ? name : "Anonymous"

status = items.length !== 0 ? "Has items" : "Empty"
```

## Nullable Conditions

Nullable values are allowed directly in condition position. The condition checks whether the condition expression itself is not `null`.

```first
displayName = user ? user.name : "Anonymous"
```

The true arm can use the non-null type.

```first
user is User or null

displayName = user ? user.name : "Anonymous"
```

A non-nullable value is not allowed directly in condition position just because it contains nullable fields.

```first
label = profile ? "Has profile" : "No profile"
// Invalid when profile itself is not nullable

label = profile.photo ? "Has photo" : "No photo"
// Valid when profile.photo is nullable
```

Nullable values keep their type when assigned to a local. The local can then be checked directly.

```first
photo = profile.photo

label = photo ? "Has photo" : "No photo"
```

Nullable booleans follow the same rule as conditionals: `false` is still a boolean value, not `null`.

```first
enabled is boolean or null

label = enabled ? "Enabled" : "Disabled or unset"
```

Use an explicit comparison when the distinction matters.

```first
label = enabled !== null ? "Set" : "Unset"
```

## Arm Types

The result type of a ternary is the shared type of its two arms.

```first
label = enabled ? "Enabled" : "Disabled"
// label is string
```

When the two arms produce different types, the result is their union or common supertype.

```first
value = useName ? name : count
// value is string or the type of count
```
