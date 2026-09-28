# Attestations

## Selection Membership

`value is Selection` tests whether a value belongs to a selection and narrows that value in the successful branch.

```first
Width is one of (16, 32)

useWidth(width is Width) ()

run(input is int) (
	useWidth(input) // Notice: membership is not established.
	if (input is Width) (
		useWidth(input) // Accepted: 16 or 32.
	)
)
```

For literal alternatives, attest exact value membership. For arrays and anonymous objects, compare deeply immutable values structurally: array order matters; objects require exactly the same fields and recursively equal values. There is no implicit coercion or user-defined equality call.

```first
Reply is one of ({ code = 1 })

run() (
	exact = { code = 1 }
	extra = { code = 1, other = true }
	a = exact is Reply // true
	b = extra is Reply // false
)
```

For type alternatives such as `Person is one of (User, Admin, null)`, use ordinary type membership for the class alternatives and exact membership for null. An attestation does not construct instances. Selections remain structurally compatible according to the values they permit; a broader source needs narrowing before use as a narrower selection.

`many of` membership follows its flag representation, and `one case of` membership follows case names and payload types. See [[11-Selection-Types]].
