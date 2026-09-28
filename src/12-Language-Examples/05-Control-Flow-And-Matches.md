# Control Flow And Matches

## Classifying A Number

This function uses sequential conditions with early returns.

```first
classify(value is int) is string (
	if (value < 0) (
		return "negative"
	)
	
	if (value == 0) (
		return "zero"
	)
	
	return "positive"
)
```

## Matching A Numeric Range

This captured match assigns a label according to the first matching arm.

```first
temperatureLabel(celsius is int) is string (
	return celsius matches (
		-50 to 0 "freezing"
		1 to 15 "cold"
		16 to 24 "comfortable"
		25 to 35 "warm"
		else "extreme"
	)
)
```

## Sharing A Match Body

Several command names intentionally select the same behavior.

```first
normalizedCommand(command is string) is string (
	return command matches (
		"q"
		"quit"
		"exit" "stop"
		
		"h"
		"help" "show help"
		
		else "unknown"
	)
)
```

## Searching With Nested Control Flow

This composed example uses a loop, a condition, and a captured match.

```first
firstPassingGrade(grades is int[]) is string (
	grades each grade index (
		label = grade matches (
			90 to 100 "excellent"
			70 to 89 "passing"
			else "retry"
		)
		
		if (grade >= 70) (
			return `{index}: {label}`
		)
	)
	
	return "none"
)
```

