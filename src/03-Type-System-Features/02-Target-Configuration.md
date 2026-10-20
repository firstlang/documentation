# Target Configuration

`declare target` selects a backend and configures the output for a space.

```first
declare target swift {
	path = "../src-swift",
	options = {}
}

declare target rust {
	path = "../src-rust",
	options = {}
}
```

The backend name is an all-lowercase identifier. Backend implementations define their own options; the language does not enumerate the available backends or standardize their option fields. Backends are included in the compiler.

## Optional Configuration

The configuration object and both fields are optional.

```first
declare target swift

declare target rust {
	options = {}
}
```

If `path` is omitted, the compiler chooses an output directory. If `options` is omitted, the backend uses its default options.

When supplied, `path` is a literal string and `options` is an object literal. Configuration values may be strings, numbers, booleans, or selections from compiler-defined `one of` types provided for configuration. Configuration does not evaluate expressions or perform computation.

## Scope and Target Selection

Targets may be declared at the project root or in spaces at any nesting depth. A target selects the declarations in its containing space and descendants, subject to nested target declarations. It creates no runtime binding.

A target declaration directly in a class or function produces a notice and has no target-selection effect. If its containing space becomes a class, that declaration becomes invalid.

Different backends declared in one logical space form its target set. Shared source targeting several backends is compiled separately for each.

```first
Common (
	declare target typescript
	declare target rust

	Record (
		constructor
		name is string
	)
)
```

A descendant without target declarations inherits the nearest enclosing target set and its complete configurations. A descendant with explicit target declarations replaces that set. Its explicit configurations do not merge with ancestor configurations; omitted option fields use backend defaults.

A space with neither explicit nor inherited targets produces no output. This is valid: a space may describe a shape without requesting generated code.

## Repeated Declarations

Several declarations for the same backend in one logical space merge their configuration objects using shallow copy-over behavior, like `Object.assign`. Later supplied fields replace earlier fields; omitted fields preserve earlier values. A supplied `options` object replaces the earlier `options` object rather than recursively merging it.

```first
declare target rust {
	path = "../src-rust"
}

declare target rust {
	options = {}
}
```

These declarations select one Rust target with the supplied path and options. The rule also applies to fragments contributing to the same logical space.

## Output Paths

`path` names the directory receiving generated source and associated build metadata. Each backend defines the layout beneath it and any subsequent build steps. It does not name the final executable.

Relative paths resolve against the First project root. Absolute paths are accepted. Normalize `.` and `..` when identifying destinations; parent-directory destinations such as `../src-rust` are allowed. Source storage and the compiler's working directory do not change path resolution.

If two spaces independently target the same directory, report a notice and suppress one competing output. Which output is suppressed is unspecified. Descendants inheriting one target configuration contribute to that output rather than creating competing destinations.

## Backend Configuration

Backend-specific options are documented with their respective backend. The following entries collect older backend-specific declarations for reassessment; their replacement option names and behavior have not yet been settled.

### Rust

| Earlier declaration | Earlier intent |
| --- | --- |
| `declare noRustStandardLibrary` | Omit the Rust standard library from output. |
| `declare external` | Preserve a function's name for foreign-function linkage, previously described using Rust's `no_mangle`. |

### TypeScript / ECMA

| Earlier declaration | Earlier intent |
| --- | --- |
| `declare noEcmaStandardLibrary` | Omit ECMA-262 built-ins from compilation output. |

GPU targeting, WebAssembly targeting, and the worker restriction also need reassessment for backend configuration. Their existing scoped forms remain listed in [[01-Declare]] until their placement and behavior are resolved. Type-local layout declarations, including packing, alignment, byte order, and selection representation, continue to describe the affected type.
