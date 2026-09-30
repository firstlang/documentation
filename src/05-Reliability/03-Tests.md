# Tests

First has no special test declaration syntax. Tests are ordinary calls into compiler-known standard modules.

```first
import Test
import Assert

Test.case("addition works", () => (
	Assert.equal(1 + 1, 2)
))
```

Tests can be grouped.

```first
import Test
import Assert

Test.group("Math", () => (
	Test.case("adds", () => (
		Assert.equal(add(1, 2), 3)
	))
))
```

The `Test` module owns test discovery, grouping, filtering, lifecycle hooks, reporting, coverage, and runner behavior. The `Assert` module owns assertions.

The compiler recognizes calls such as `Test.case` and `Test.group` for test discovery, editor display, release stripping, and test entry-point behavior.

Program-owned startup functions do not run for test entry points. Imported wrapper startup still runs, because imported modules must still be made valid before their APIs are used.
