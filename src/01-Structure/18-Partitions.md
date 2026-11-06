# Partitions

Partitions organize a program by criticality. Spaces describe responsibilities; partitions provide another axis, separating the architectural shape of those responsibilities from progressively deeper implementation detail.

Merging exists to connect intentional contributions across partition levels. Repeating a declaration within one partition is usually misuse: if its parts need to remain separate, they should occupy different partitions. Same-partition duplicates receive notices rather than silently becoming overloads or combined implementations.

## The Architectural Root

An unpartitioned contribution belongs to the implicit architectural root. No declaration is needed to select this root.

```first
ServiceManager (
	getString() returns string
)
```

Named partitions form a tree below that root. `logic` is one level below it; `logic internals` selects `internals` below `logic`. Paths are space-separated and may have arbitrary depth.

```first
ServiceManager (
	declare partition logic
	// Implementation contribution.
)

ServiceManager (
	declare partition logic internals
	// Deeper implementation contribution.
)
```

Partition names select levels of detail; they do not create namespaces, subclasses, or runtime instances.

## Partition Blocks

A parenthesized block may apply a partition to several declarations:

```first
(
	declare partition logic

	ServiceManager (
		// ServiceManager's logic contribution.
	)

	WorkspaceProvider (
		// WorkspaceProvider's logic contribution.
	)
)
```

This block introduces no lexical scope or runtime scope. Each contribution retains its ordinary structural owner. The selected partition path is relative to the enclosing partition context.

Files do not determine membership. Contributions may be stored together or in separate files without changing their logical owner or partition path.

## Layered Merging

Contributions to the same logical declaration at different partition levels form one declaration. Matching uses the logical parent, name, and declaration kind. The partition records where each contribution belongs in the criticality tree.

Merging preserves the declaration's identity. A deeper function contribution does not introduce an overload or replace the ancestor declaration.

Two declarations of the same function within the same partition are duplicates, even if their signatures agree. Changing files or repeating a partition block does not make such duplication valid. Ordinary declarations in distinct lexical scopes retain their separate identities.

## Visibility

A partition can refer to its own declarations and declarations in its ancestors. It cannot refer to declarations introduced only in descendants or siblings.

```first
ServiceManager (
	validate() (
		// Visibility notice: this helper exists only below this boundary.
		validateInner()
	)
)

ServiceManager (
	declare partition implementation
	validateInner() (
		// The architectural declaration is visible here.
		validate()
	)
)
```

The calls illustrate name visibility, not a recommended execution pattern.

Sibling communication goes through declarations, contracts, or signals in the shared parent. There is no partition friendship mechanism. A parent-level function can receive its implementation from a lower partition without making the lower partition's private implementation names visible to the parent.

## Function Implementations And Advice

A merged function may have at most one bodyless architectural declaration and at most one value-producing implementation body. When an architectural declaration exists, the implementation must declare exactly the same return type.

Additional function contributions must omit a return type and produce no value. These void bodies act as advice: they execute before the implementation body. They belong to different partitions under the same function identity.

```first
ServiceManager (
	getString() returns string
)

ServiceManager (
	declare partition logic
	getString() (
		console.log("Preparing")
	)
)

ServiceManager (
	declare partition logic internals
	getString() returns string (
		return "Ready"
	)
)
```

Calling `ServiceManager.getString()` prints `Preparing`, then returns `Ready`. These contributions form one callable function. The architectural signature contributes no executable statements.

The compiler constructs the partition tree and orders advice bodies by depth-first traversal. Sibling branches follow declaration order. All advice executes before the value-producing implementation, regardless of that implementation's position in the tree. Body merging does not rebind names: each contribution resolves references using its own partition visibility.

Declaration order across files comes from the project's source-assembly order. The exact file-ordering mechanism remains to be specified; partition semantics do not introduce a separate file-ordering policy.

## Type Refinement

A deeper contribution may preserve or narrow the type exposed by an ancestor; it may not widen that contract. Function implementation return types have the stricter rule above: the declared implementation return type must match the architectural return type exactly. Returning a particular member of a union does not require narrowing the declaration:

```first
ServiceManager (
	getValue() returns string or int
)

ServiceManager (
	declare partition implementation
	getValue() returns string or int (
		return "Ready"
	)
)
```

Void advice contributes behavior to the existing function without changing its public return type. Omitting a return type on advice does not turn the merged function into a void function.

## Views And Compilation

The compiler and language service can inspect a selected partition with its ancestor declarations and required interface dependencies. Descendant implementations need not be loaded to understand the architectural view. Lookup never searches hidden descendants or siblings to repair an unresolved reference.

An editor view that hides lower partitions does not change the complete program's behavior. Compiling an incomplete partition view is distinct from executing the fully assembled implementation; missing bodies retain ordinary incomplete-function behavior. Files and target backends remain independent of partition selection.

## Notices

The compiler reports notices for same-partition duplicate declarations, references across a forbidden visibility boundary, widened ancestor contracts, implementation return types that differ from architectural declarations, and multiple value-producing implementations of one function.

Partition merging does not silently select the first or last implementation to resolve these conflicts. Construct-specific recovery and the remaining details of advice control flow follow their respective language rules as those rules are specified.
