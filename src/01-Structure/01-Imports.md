# Imports

Imports make built-in modules and wrapper packages available inside a space or class.

First intentionally has a small import syntax. Ordinary package imports use canonical names rather than strings or paths. Named, default, namespace, type-only, and dynamic TypeScript imports are not supported.

## Name Categories

First has three relevant categories of names:

- Standard built-ins: `Math`, `console`, `Array`
- Module built-ins: `Fs`, `Path`, `Net`
- Wrapper packages: `Serde`, `Postgres`, `Sqlx`

Standard built-ins are always available. Module built-ins and wrapper packages must be imported before use.

A wrapper package is a First-facing module generated around a Rust crate. First code imports the wrapper rather than accessing the crate directly. The wrapper adapts the crate's API, types, ownership behavior, documentation, startup metadata, and Cargo features to First.

## Module Imports

The basic import form uses a module's canonical name:

```first
import Postgres
```

An alias changes the local binding:

```first
import Postgres as MyPostgres
```

The alias may be any valid non-reserved First identifier. UpperCamelCase is conventional but not mechanically required. An alias affects only lexical lookup; it does not rename the package or its API.

An import may specify Cargo features and an exact package version:

```first
import Sqlx features postgres mysql sqlite version 1.2.3
```

Features are whitespace-separated. When an alias, features, and a version are all present, they occur in that order, with the version last:

```first
import Sqlx as Database features postgres mysql sqlite version 1.2.3
```

## Declared Import Policy

A `declare import` establishes inherited package policy without introducing a module binding:

```first
declare import Sqlx features postgres mysql sqlite version 1.2.3
```

The declaration applies in its containing space or class and in nested spaces and classes. A matching import inherits any version or features it does not state itself:

```first
declare import Sqlx features postgres version 1.2.3

DataAccess (
	import Sqlx
)
```

Import-local options take precedence over the nearest visible `declare import`. A declaration does not initialize or include a package unless a matching import exists.

## Version Resolution

Versions are exact semantic versions. A version written on an import or inherited from `declare import` selects that wrapper and its underlying crate version.

Bare imports are valid:

```first
import Postgres
```

The package manager first selects the newest compatible wrapper version already available in its local cache. If no compatible cached version exists, it selects and downloads the newest compatible non-yanked registry version. The editor then rewrites the import with the exact selected version:

```first
import Postgres version 1.2.3
```

The interval before that rewrite is intentionally nondeterministic. Canonical persisted code normally contains the resolved version, and editor-managed lock state records the complete transitive dependency graph.

Different spaces and classes may explicitly import different versions of the same package. Each version produces a distinct module value and distinct Rust types. Those types are not interchangeable merely because their names match. One space or class cannot import two versions of the same package.

## Cargo Features

Wrapper metadata maps its First feature names to Cargo features. Wrapper-facing feature names are ordinary First identifiers; for example, a Cargo feature named `native-tls` can be exposed as `nativeTls`. Features required unambiguously by API use are inferred. The editor writes inferred features into the import or applicable `declare import` automatically.

Some features select behavior that API use cannot reveal, such as a TLS implementation, runtime integration, database backend, compression format, macro support, or bundled native library. Those features must be selected explicitly or through an editor decision.

Cargo unions all requested features for one package version across the final dependency graph. First follows that rule even when the feature requests originate in different lexical spaces or classes. Features for different package versions remain separate. Mutually incompatible features produce a notice and dependency-resolution recovery.

## Canonical Package Names

Ordinary wrapper names derive from Cargo package names. Hyphens and underscores divide words, and wrapper metadata applies First's .NET-style casing:

```text
tokio-postgres -> TokioPostgres
serde_json     -> SerdeJson
html-emitter   -> HtmlEmitter
sql-service    -> SqlService
ip-location    -> IPLocation
sqlx           -> Sqlx
```

Two-letter initialisms remain uppercase. Longer initialisms use ordinary word casing. Wrapper metadata preserves recognized initialisms when the Cargo spelling alone is insufficient to recover them.

Built-in module names are reserved. In the rare case where a Cargo package collides with a built-in or otherwise cannot be named unambiguously, its exact Cargo name may be quoted and must be aliased:

```first
import Path
import "path" as CratePath version 1.2.3
```

Quoted package names are an escape hatch, not the ordinary package syntax. They undergo the same cache-first version resolution and editor pinning as canonical package imports.

## Placement And Scope

Imports and `declare import` forms must precede every other semantic member of their containing space or class. Comments and anchors may precede or appear between them.

The implicit project-root space and every explicit space and class follow the same rule. Functions, startup functions, control-flow blocks, and other runtime constructs cannot contain imports.

An import in the implicit root space is visible project-wide. An import in an explicit space or class belongs to that logical merged declaration and is visible to all of its structural contributions and descendants. Source files do not create lexical scopes and are not included through import syntax. Aliases and inherited imports follow the structural containment hierarchy.

A nested declaration may shadow an inherited import binding. A conflict in the same lexical scope still produces a notice. Imports remain lexical bindings rather than members, so an imported name cannot be reached through an external instance or space path and is not re-exported.

Within one space or class, the first import of a package creates its module binding. A later import of that package produces a notice and creates neither another alias nor another startup entry, even when it requests a different version.

## Module Values

A module import creates a runtime module value:

```first
import Postgres version 1.2.3

entries = Object.entries(Postgres)
```

Module values may be assigned, passed, returned, stored, and inspected. Each resolved wrapper package and version has one shared module value per program, so imports and aliases of that version refer to the same value.

Module values are frozen. Code cannot add, replace, or remove their members. Wrapped functions and mutable Rust statics may still modify their own underlying state through their defined APIs.

Generated wrappers expose Rust APIs through wrapper generation rather than source access declarations. Exposure is best-effort, and the exact static and runtime surface may evolve as wrapper generation improves. The generated wrapper and its documentation define what a particular package version exposes, including what appears through runtime inspection.

An imported module binding cannot be exported or re-exported. Only items explicitly defined within a space or class may be exported from it.

## Startup

Wrapper metadata may define a generated startup function. Every valid wrapper imported anywhere in the compiled project participates in eager startup, even when its binding is unused or appears inside an otherwise unused space or class. Lexical scope controls visibility, not initialization. The same behavior applies to test entry points.

Startup functions are synchronous and cannot use `await`. A module that requires asynchronous or fallible initialization must expose an ordinary function for project startup function code to call explicitly.

Startup functions should be independent. When static analysis finds that one startup function accesses another imported module, the compiler records a dependency and topologically orders the generated calls. Independent startup functions have no semantic order, and programs must not rely on incidental emitted order.

Each wrapper package version starts at most once. A startup dependency cycle produces notices; every startup function in the cyclic component is short-circuited through `unknowable` recovery.

Startup functions cannot return bombs. Attempting to do so produces a notice. If a bomb occurs at runtime, execution of that startup function stops.

The compiler injects wrapper startup calls into the final generated Rust `main` before running project startup functions. Rust library crates do not provide the executed program entry point; the generated First wrapper supplies the startup hook and metadata.

## Invalid Imports And Recovery

First reports invalid imports with notices rather than compilation errors. A malformed, misplaced, unavailable, or unresolvable module import creates an internal `unknowable` module placeholder. Values derived from it are unknowable, and affected statements are short-circuited so the rest of the program remains executable.

An invalid wrapper schedules no startup. `unknowable` is compiler recovery state, not a user-expressible type or a value intended for shipped code. Its propagation rules are defined in [Unknowable Recovery](../../03-Compiler/01-Front-End/01-Unknowable-Recovery.md).

Quoted source paths and source globs are not import forms. First source files use the `.first` extension, but files remain a storage detail rather than inclusion units.

## Unsupported TypeScript Forms

First does not support TypeScript-style import forms or dynamic imports:

```ts
import { readFile } from "fs"
import Fs from "fs"
import * as Fs from "fs"
import type { Thing } from "thing"
const Fs = await import("fs")
import("fs")
```

## Package Management And Rust Lowering

An import is both a lexical binding and a direct wrapper dependency declaration. The compiler and package manager resolve it, consult their caches and registry, generate the required Cargo project information in the build-output directory, and lower calls through the generated wrapper.

Package discovery, downloading, locking, feature resolution, wrapper generation, platform selection, and Rust project generation are package-manager responsibilities. They never require a second hand-authored dependency manifest.

The software bill of materials is an editor concern. The editor derives BOM, dependency, version, feature, license, and supply-chain views from imports, wrapper metadata, and resolved lock state rather than representing the BOM as ordinary source code.
