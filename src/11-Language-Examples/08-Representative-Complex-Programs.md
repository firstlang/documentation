# Representative Complex Programs

These examples apply realistic pressure to the editor layout without relying on malformed source, code golf, or intentionally poor style.

## Summarizing An Order

This example combines a class, an array of instances, numeric accumulation, branching, and multiline interpolation.

```first
OrderLine (
	name is string
	quantity is int
	unitPrice is int
	
	constructor(name is string, quantity is int, unitPrice is int) (
		this.name = name
		this.quantity = quantity
		this.unitPrice = unitPrice
	)
	
	total() is int (
		return quantity * unitPrice
	)
)

orderSummary(lines is OrderLine[]) is string (
	total is var int = 0
	
	lines each line (
		total += line.total()
	)
	
	return `Line count: {lines.length}\n` +
		`Order total: {total}`
)
```

## Grouping Measurements

Nested arrays arise naturally when a program groups sequential readings into batches.

```first
groupMeasurements(values is int[], groupSize is int) is int[][] (
	groups is editable int[][]
	current is var editable int[]
	
	values each value index (
		current.push(value)
		position = index + 1
		
		if (position % groupSize == 0) (
			groups.push(current)
			current = []
		)
	)
	
	if (current.length > 0) (
		groups.push(current)
	)
	
	return groups
)
```

## Producing A Diagnostic Report

This program mixes matching, nested control flow, member calls, and accumulated text while preserving useful breakpoint locations.

```first
diagnosticReport(readings is int[]) is string (
	report is var string = "Readings"
	
	readings each reading index (
		severity = reading matches (
			0 to 49 "normal"
			50 to 79 "elevated"
			80 to 100 "critical"
			else "invalid"
		)
		
		if (severity != "normal") (
			report += `\n{index}: {reading} — {severity}`
		)
	)
	
	return report
)
```

## Transforming A Grid

This example creates a meaningful level of nesting through a two-dimensional data transformation.

```first
scaleGrid(grid is int[][], factor is int) is int[][] (
	result is editable int[][]
	
	grid each sourceRow (
		resultRow is editable int[]
		
		sourceRow each value (
			scaled = value * factor
			resultRow.push(scaled)
		)
		
		result.push(resultRow)
	)
	
	return result
)
```
