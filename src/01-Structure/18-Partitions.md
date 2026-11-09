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

## Function Contributions And Execution

A merged function may have at most one bodyless architectural declaration and multiple executable contributions in different partitions. Contributions form one function body, not separate function calls. The architectural signature contributes no executable statements.

The compiler orders executable contributions by depth-first traversal of the partition tree. Sibling branches follow declaration order. Each contribution may supply a result, let a bomb escape, or complete without supplying a value (`void`). Completing a contribution without a value continues execution into the next contribution; it does not return from the merged function.

```first
ServiceManager (
	getString(early is boolean) returns string
)

ServiceManager (
	declare partition logic
	getString(early is boolean) returns string (
		if (early) (
			return "Early"
		)
	)
)

ServiceManager (
	declare partition logic internals
	getString(early is boolean) returns string (
		return "Ready"
	)
)
```

When `early` is true, the first contribution returns `"Early"` and the second contribution does not execute. Otherwise, the first contribution falls through and the second returns `"Ready"`.

Several contributions may produce values. The first executed return targeting the function completes the merged call and skips the remaining contributions. A bomb escaping the function likewise completes the call and skips the remaining contributions.

A function-targeting bare `return` also completes the entire call, following the ordinary typed zero-value rule and return-site notice. It does not merely end the current contribution. A return captured by a nested expression retains its ordinary destination; see [[../04-Control-Flow-And-Evaluation/06-Return]].

To skip a section without completing the call, place it in an explicit standalone scope and use `break` to exit that scope. Neither a contribution body nor the merged function body is a break target. Existing break and return target rules remain unchanged.

Zero-value fallback applies only after execution reaches the end of the whole merged body without supplying a result, not at the end of each contribution. If the declared result type does not permit `void`, the merged function supplies that type's ordinary zero value. Contribution-level `void` does not add `void` or implicit `null` to the caller-visible result type.

Body merging does not rebind names: each contribution resolves references using its own partition visibility. Declaration order across files comes from the project's source-assembly order. The exact file-ordering mechanism remains to be specified; partition semantics do not introduce a separate file-ordering policy.

## Cross-Partition Contracts

Function contributions preserve the architectural parameter list and parameter types. Deeper contributions may not add parameters, remove parameters, or narrow or widen their types. A change to the required inputs belongs in the architectural declaration.

The architectural result type defines the successful result contract. A contribution may omit its return annotation or repeat the architectural result type; it may not redeclare a narrower or wider successful result type. Returning a compatible subtype or a member of a declared union does not change that contract.

For an architectural result type `T`, each contribution may produce a value compatible with `T`, complete without a value (`void`), or let a `bomb` escape. These are the only permitted implementation-level extensions. In particular, a contribution may not add `null` or an unrelated successful result alternative unless the architectural type already permits it.

Bombability is inferred across all executable contributions. An escaping bomb expands the caller-visible result from `T` to `T or bomb` without requiring that alternative to be written in the architectural declaration. Bombs handled internally do not by themselves make the merged function bombable. Unlike contribution-level `void`, an escaping bomb is a result of the complete call.

Other typed declarations retain the ordinary refinement rule: a deeper contribution may preserve or narrow an ancestor type, but may not widen it.

## Views And Compilation

The compiler and language service can inspect a selected partition with its ancestor declarations and required interface dependencies. Descendant implementations need not be loaded to understand the declared architectural contract. Determining inferred bombability requires the executable contributions; an architectural declaration alone does not establish that a function cannot bomb. Lookup never searches hidden descendants or siblings to repair an unresolved reference.

Compiling an incomplete partition view is distinct from executing the fully assembled implementation; missing bodies retain ordinary incomplete-function behavior. Files and target backends remain independent of partition selection.

## Notices

The compiler reports notices for same-partition duplicate declarations, references across a forbidden visibility boundary, forbidden changes to architectural parameters or successful result types, widened non-function ancestor contracts, incompatible returned values, and multiple bodyless architectural declarations of one function.

Partition merging does not silently select the first or last implementation to resolve these conflicts. Construct-specific recovery follows the respective language rules.
