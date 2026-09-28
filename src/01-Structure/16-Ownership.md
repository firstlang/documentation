Ownership controls whether stored references keep their targets alive.

First uses gradual ownership. The compiler infers ordinary ownership, borrowing, shared ownership, and the storage needed to implement them. First programmers do not write Rust lifetimes, borrow annotations, reference-counting types, or interior-mutability wrappers.

Ownership modifiers are needed only when the programmer must express lifetime behavior that inference cannot choose or acknowledge a potentially leaking ownership cycle.

```first
Node (
	constructor
	
	parent is weak Node
	peer is strong Node
)
```

`weak` and `strong` are type-side modifiers. They are not declaration modifiers and do not use ownership groups.

Typist may hide ownership modifiers in ordinary views. It may expose them as badges, icons, or in a dedicated ownership view. The modifiers remain part of the canonical program information even when a view does not render them as text.

## Inferred Ownership

An unannotated stored reference uses inferred ownership.

The compiler may represent an inferred reference as an owned value, a borrow, a uniquely allocated value, or a reference-counted value. It may also generate the machinery needed for shared mutation. The representation is an implementation detail and may change without changing the observable behavior of the program.

```first
Tree (
	constructor
	
	root is Node
)
```

The absence of an ownership modifier does not mean that the generated Rust field has one particular representation. It directs the compiler to choose a representation that preserves First semantics and produces valid Rust.

When a more precise representation cannot be used, the compiler may conservatively fall back to reference counting. Crossing a Worker boundary similarly permits the compiler to select thread-safe shared ownership rather than single-threaded shared ownership.

## Weak References

A `weak` reference does not keep its target alive.

```first
Child (
	constructor
	
	parent is weak Parent
)
```

A weak reference is inherently nullable. It may have no target before assignment, and its target may disappear after the target's strong owners release it. Nullability cannot be removed from a weak reference.

Reading an expired weak reference produces `null`. Operations through a weak reference use First's shield behavior. A shield indicates that an operation or statement may short-circuit because the target is null. This includes reads, calls, and assignments through a nullable chain.

Weak ownership is part of the field's type and observable semantics. Assigning another value to the field does not change its ownership classification.

Only reference-class types may be weak. Value types cannot be weak because their values are stored directly rather than referred to through independently owned object identity.

## Explicit Strong References

`strong` acknowledges that a reference intentionally participates in a potential strong ownership cycle.

```first
Node (
	constructor
	
	connections is Array(strong Node)
)
```

All stored references keep their targets alive unless they are weak. `strong` therefore does not introduce stronger runtime behavior than an equivalent unannotated strong reference. It records that the programmer accepts the consequences of the cycle, including the possibility that its values never get destroyed.

When the compiler finds a potential strong ownership cycle without an explicit `strong` edge, it still compiles the program. It uses reference counting where needed and generates a notice asking whether the cycle is intentional. Adding `strong` to at least one participating edge acknowledges the strongly connected region and silences that notice.

An unnecessary `strong` modifier is permitted. The compiler generates a notice when it can determine that the modifier does not acknowledge a potential cycle.

Explicit strong ownership does not form a distinct assignable type. It is ownership metadata carried with the type declaration. Adding or removing an unnecessary `strong` modifier does not change type compatibility.

## Recursive Types

Recursive type relationships are permitted.

```first
Parent (
	constructor
	
	children is Array(Child)
)

Child (
	constructor
	
	parent is weak Parent
)
```

The complete type graph in this example is recursive, but its strong ownership graph is acyclic because the reference from `Child` to `Parent` is weak. It does not require an explicit `strong` acknowledgement.

If every edge completing a potential type-level ownership cycle is strong, at least one participating edge must be marked `strong` to acknowledge the cycle without a notice.

## Containers

Containers own their storage normally. Ownership modifiers on generic arguments classify the values stored in the corresponding generic positions.

```first
Graph (
	constructor
	
	nodes is Array(Node)
	observers is Array(weak Node)
	relationships is Map(Node, weak Node)
)
```

In this example, `nodes` stores strong `Node` references, `observers` stores weak `Node` references, and `relationships` stores strong keys and weak values. The ownership of one generic position does not implicitly change another position.

An ownership modifier applied to the container type itself classifies the container reference rather than its elements.

```first
Cache (
	constructor
	
	sharedEntries is weak Map(string, Entry)
)
```

Here the `Map` object is weakly referenced. Its keys and values retain the ownership behavior declared by `Map` and its generic arguments.

Unannotated generic positions use the container type's ordinary ownership behavior and gradual ownership inference. Programmers do not need to annotate generic arguments unless they intend weak behavior or must acknowledge a detected cycle.

## Placement

Ownership modifiers may classify stored fields.

```first
Link (
	constructor(target is weak Node) (
		this.target = target
	)
	
	target is weak Node
)
```

In this example, `target` is an ordinary weak field. Constructor parameters are not promoted by access markers.

Ownership modifiers do not classify ordinary parameters, local variables, or closure captures. The compiler infers their Rust ownership and capture representations. These decisions may be shown in a diagnostic or ownership view without becoming source-level annotations.

## Modifier Order

When mutability and ownership modifiers are both present, their canonical order is mutability, ownership, then the type.

```first
Selection (
	constructor
	
	current is var weak Node
)
```

## Assignment

Ownership belongs to the stored field declaration, not to an individual assigned value.

Assigning a value to an inferred or explicit strong field makes the field retain the target according to its generated representation. Assigning a value to a weak field records a weak reference and does not keep the target alive.

The compiler inserts any Rust moves, clones, upgrades, downgrades, borrows, or generated wrappers needed to preserve these semantics. These operations are not part of First's surface ownership model.

## Inheritance

Inherited fields retain their original ownership classification. A child class cannot reclassify an inherited field from weak to strong or from strong to weak.

Weak ownership affects nullability and lifetime behavior, so it is part of the effective public type signature. Explicit `strong` acknowledgement is retained as ownership metadata but does not affect assignment compatibility.

Typist may omit ownership information from ordinary API views when it is not relevant to the reader. An ownership-focused view must be able to show the complete effective ownership signature, including inherited fields.

## Ghosts and Cycles

Ghosts run only when a value is actually destroyed.

A strong reference cycle may keep every reference count above zero indefinitely. If that happens, the values leak and their ghosts do not run. A ghost cannot break the cycle that prevents its own execution.

Code may explicitly disconnect a reachable graph before releasing it. If breaking its strong edges allows the reference counts to reach zero, destruction proceeds normally and the corresponding ghosts run. First does not require the compiler to prove that an acknowledged strong cycle will eventually be broken.

## Notices and Recovery

The compiler generates a notice when:

- A potential strong ownership cycle has no explicit `strong` acknowledgement.
- `strong` is applied where the compiler can determine that no potential cycle exists.
- `weak` is applied to a value type.
- An ownership modifier is applied to an unsupported declaration.
- Conflicting ownership modifiers classify the same type position.

First still compiles after these notices.

For an unacknowledged cycle, the compiler conservatively uses reference counting where needed. For `weak` on a value type or an unsupported declaration, the compiler ignores the modifier. When modifiers conflict, the last modifier wins for recovery.

## Rust Output

First does not guarantee a particular Rust representation for an ownership classification. The compiler may use ordinary values, borrows, `Box`, `Rc`, `Arc`, `Weak`, interior-mutability containers, locks, or other representations that preserve the program's observable semantics.

A single-threaded parent and child relationship may have the following Rust shape:

```rust
struct Parent {
	children: Vec<Rc<Child>>,
}

struct Child {
	parent: Weak<Parent>,
}
```

This example illustrates a plausible lowering, not a required output form. The compiler may select another valid representation when its analysis provides a more precise option.
