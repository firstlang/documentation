# Strings And Text

## Greeting A User

This example uses a default parameter and interpolation.

```first
greeting(name is string, punctuation is string = "!") is string (
	return `Hello, {name}{punctuation}`
)
```

## Constructing Multiline Text

Both string delimiter forms support multiline text. Backticks allow interpolation; the shared leading whitespace is trimmed, and whitespace-only boundary lines are excluded.

```first
addressLabel(name is string, city is string, country is string) is string (
	return `
		{name}
		{city}
		{country}
	`
)
```

## Describing A List

This function combines array iteration, branching, mutation, and interpolation.

```first
bulletList(items is string[]) is string (
	result is var string = "Items:"
	
	items each item index (
		position = index + 1
		result += `\n{position}. {item}`
	)
	
	return result
)
```

## Formatting A Small Report

This denser string example keeps each meaningful operation on its own line.

```first
scoreReport(name is string, scores is int[]) is string (
	total is var int = 0
	
	scores each score (
		total += score
	)
	
	count = scores.length
	average = total / count
	
	return `Student: {name}\n` +
		`Scores: {scores}\n` +
		`Average: {average}`
)
```
