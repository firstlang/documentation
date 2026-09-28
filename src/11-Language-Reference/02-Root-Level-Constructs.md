# Root-Level Constructs

The project root is the language's only unnamed space. It accepts imports, `declare` forms, functions, values, types, startup functions, child spaces, and child classes.

## Imports

Canonical module imports have this shape:

```text
import ModuleName [as LocalName] [features Feature...] [version Semver]
```

The quoted Cargo-name escape hatch requires an alias:

```text
import "cargo-name" as LocalName [features Feature...] [version Semver]
```

Inherited wrapper policy has this shape and creates no binding:

```text
declare import ModuleName [features Feature...] [version Semver]
```

Imports and `declare import` forms precede all other semantic root members. Comments and anchors may precede or occur between them.

## Selections

```first
Width is one of (16, 32)
Person is one of (User, Admin, null)
Permission is many of (read, write)
Message is one case of (text(value is string), closed())
Extended is Base or one of (extra)
Combined is First or Second or Third
```

Selection composition uses `or`; selection bodies do not support spread. `one of` and `many of` bodies contain no author-defined functions. The old `one value of` form is retired.
