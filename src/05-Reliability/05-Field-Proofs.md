# Field Proofs

> **Status:** Provisional. The settled semantics below define the known field-proof model. The final section records unresolved design choices and proposed resolutions.

Field proofs are mutation-aware proofs. They certify that a named issuing function established evidence for a particular field state, and they remain valid only while every contributing field retains that state.

A field proof is declared with `field proof`:

```first
HighScore is field proof (
)
```

Field proofs extend the ordinary proof system defined in [[04-Proofs]]. Unless this document says otherwise, they inherit ordinary proof rules for nominal identity, visibility, relationship arguments, issuing authority, pure and attached applications, exact function contracts, control-flow availability, runtime erasure, notices, and tooling provenance.

# Settled design

## Relationship to ordinary proofs

Ordinary proofs are durable proof-of-passage wristbands. Mutation never invalidates them.

Field proofs express a different guarantee: the evidence remains applicable to the current state of particular fields. Mutating a contributing field invalidates the field proof. Mutating an unrelated field does not.

```first
HighScore is field proof (
)

verifyHighScore(user is User) is User with HighScore 💣 (
	if user.score > 50 (
		return user with HighScore
	)
	
	throw "Score is too low"
)
```

Conceptually, successful issuance associates:

```text
HighScore on User#17
depends on User#17.score
```

Therefore:

```first
user.name = "Paul"
// HighScore survives when name is not a contributor.

user.score = 0
// HighScore is invalidated.
```

The compiler does not prove that `score > 50` means `HighScore`. The issuing function defines that meaning. Field analysis determines only which mutations may make the issued evidence stale.

## Field binding

Every field proof is bound to a dependency set consisting of fields on:

- its carrier
- runtime objects supplied as relationship arguments
- objects reached through supported field paths
- prerequisite field proofs used during issuance

The dependency set is compile-time metadata. First source cannot inspect, enumerate, compare, or modify it.

Field identity is distinct from field contents. The compiler tracks which stored locations contributed, not a dependent-type value such as “score equals 51.”

## Invalidation

A field proof becomes unavailable after a contributing field may have been mutated.

Invalidation is permanent for that evidence instance. Restoring the previous runtime contents does not restore the proof. An authorized issuing function must establish new evidence.

Mutation of a field outside the dependency set does not invalidate the proof.

Invalidation follows aliases and calls whenever the compiler can determine that they may write a contributing field. The exact analysis boundary remains provisional below.

## No runtime representation

Field proofs are fully erased, like ordinary proofs.

They introduce no runtime proof object, dependency table, observer, mutation hook, version counter, allocation, parameter, or return value. Invalidation is a compile-time fact derived by static analysis.

Field proofs cannot participate in attestations, runtime tests, branching, reflection, equality, or inspection.

## Issuing authority

Field proofs inherit ordinary issuing authority:

- one source-provenance issuing site by default
- multiple sites only when federation is permitted
- repeated execution does not create sites
- propagation is not issuance
- every successful return exactly matches its declared proof contract

Field-proof issuing functions require additional restrictions so the compiler can determine a sound dependency set. The precise restrictions remain open below.

## Attached and relationship forms

A field proof may be attached to a carrier:

```first
user with HighScore
```

It may relate that carrier to other runtime objects:

```first
ValidFor is field proof (
	array is Array
)

index with ValidFor(array)
```

In that case, dependencies may include supported fields on both the carrier and the related objects.

A field proof may also be pure when its field dependencies originate entirely from related runtime objects. Whether zero-parameter pure field proofs are meaningful remains an open question.

## Explicit re-establishment

The compiler never silently restores an invalidated field proof.

```first
user.score = 75
user = verifyHighScore(user) 💣 throw
```

Typist may explain the invalidation, navigate to the issuer, and offer an explicit AI-generated repair. Required arguments, runtime work, and bomb handling remain visible in source.

## Runtime and external state boundary

Field proofs can react only to mutations visible to the First compiler.

They cannot remain synchronized with a database, clock, network service, environment variable, foreign global, or other independently changing state without runtime machinery. The intended issuer restrictions exclude such dependencies.

Facts about historical passage through an external check belong to ordinary proofs. Facts requiring current external truth require runtime rechecking or runtime capability objects.

## Appropriate uses

Field proofs are intended for current-state invariants over mutable First data:

- an index remains valid for a mutable collection
- a graph remains acyclic
- a collection remains sorted or deduplicated
- a transfer remains authorized for its current fields
- a buffer remains initialized under operations that may reset it
- a mutable object remains validated against related objects

Ordinary proofs remain preferable for immutable, historical, monotonic, or externally checked facts that do not require compiler-visible invalidation.

# Open design ambiguities

The following decisions remain provisional. Leave `agree` unchanged to accept a proposal, or replace it with the desired rule.

## 1. Federated field-proof syntax

**Ambiguity:** The declaration syntax `Name is field proof` is settled, but the interaction with multiple issuing sites is not.

**Proposed resolution:** Permit `Name is federated field proof (parameters)`. Federation changes only issuing-site cardinality, exactly as it does for ordinary proofs; it does not change dependency inference or invalidation.

agree

## 2. How dependencies are declared

**Ambiguity:** A field proof must identify contributing fields, but it remains undecided whether authors write that set or the compiler derives it.

**Proposed resolution:** Infer dependencies from the issuing function rather than adding a dependency list to source. Typist displays the inferred set at the establishment expression and in the proof's audit view. When inference cannot remain sound within the supported analysis boundary, issuance receives a prominent notice rather than silently choosing an incomplete set.

agree

## 3. Issuer purity requirement

**Ambiguity:** The issuing function probably needs stronger restrictions than an ordinary proof issuer, but the required semantic contract is unsettled.

**Proposed resolution:** A field-proof issuer must be `pureFunctional` apart from returning or throwing a bomb. Given the same explicit inputs and field state, it must produce the same success or bomb result and cannot perform I/O, access time, randomness, ambient state, or mutable globals. Idempotence follows from this deterministic purity.

agree

## 4. Issuer termination and totality

**Ambiguity:** Referential transparency does not determine whether validation may loop forever or whether `totalFunctional` is required.

**Proposed resolution:** Do not require totality initially. A field-proof issuer may fail to terminate like another pure function, but every successful return must satisfy its exact proof contract. Typist may recommend `totalFunctional` for public security-sensitive issuers.

agree

## 5. Calls from issuing functions

**Ambiguity:** A pure issuer may call helper functions, but unrestricted transitive analysis risks recreating a whole-program effect system.

**Proposed resolution:** An issuer may call only compiler-visible First functions already proven `pureFunctional`, plus compiler intrinsics with trusted dependency summaries. Dependency inference follows those calls transitively. Dynamic, foreign, unresolved, or effectful calls are forbidden in a field-proof issuer.

agree

## 6. Direct versus nested field paths

**Ambiguity:** The compiler can easily observe `user.score`, but nested access such as `user.account.profile.score` introduces replacement and aliasing questions.

**Proposed resolution:** Track stable dependency paths through stored fields. A dependency on `user.account.profile.score` is invalidated by writing `score`, replacing `profile`, replacing `account`, or replacing the carrier reference. If an intermediate access is computed, foreign, weak, or otherwise unstable, the establishment is outside the initial supported subset.

agree

## 7. Collections and dynamic access

**Ambiguity:** Validation may read collection elements, ranges, lengths, or dynamically chosen keys, for which a precise field set may be unbounded.

**Proposed resolution:** Treat any read through a collection-valued field as dependency on that complete field, including its length, membership, ordering, keys, values, and element contents. Any mutation reachable through that collection field invalidates the proof. Element-level dependency precision is deferred.

agree

## 8. Computed properties and getters

**Ambiguity:** A field-proof issuer may read a property whose getter performs arbitrary work or reads other fields.

**Proposed resolution:** Follow a getter only when it is compiler-visible and proven pure under the same issuer restrictions; include its transitive stored-field reads. Otherwise the getter is forbidden in a field-proof issuer.

agree

## 9. Relationship-object dependencies

**Ambiguity:** A field proof may relate its carrier to other runtime objects, but it is unclear whether reads from those objects contribute to invalidation.

**Proposed resolution:** Reads from relationship arguments contribute dependency paths exactly like reads from the carrier. Mutation of any recorded field path on any related object invalidates the field proof; unrelated fields do not.

agree

## 10. Proof-bearing primitive inputs

**Ambiguity:** Ordinary relationship proofs may accept a primitive already carrying another proof, but field proofs cannot observe fields on that primitive.

**Proposed resolution:** Permit proved primitive relationship inputs as opaque prerequisites, but do not derive field dependencies from their scalar contents. If the outer field proof's success depends on inspecting the primitive value, the issuer is outside the initial supported subset.

agree

## 11. Pure field proofs

**Ambiguity:** A pure proof has no carrier, so a field proof can be mutation-aware only through related runtime objects.

**Proposed resolution:** Permit pure field proofs only when they declare at least one reference relationship parameter. Their dependency paths originate from those related objects. A zero-parameter pure field proof is invalid because it has no field state to track and should be an ordinary proof.

agree

## 12. Self-proving methods

**Ambiguity:** Instance methods may establish ordinary proofs on `this`, but exact dependency behavior for `this with FieldProof` is not yet stated.

**Proposed resolution:** Permit named, statically dispatched self-proving methods. Reads of `this` fields determine carrier dependencies, and reads of explicit relationship arguments determine related-object dependencies. The method must satisfy every field-issuer restriction.

agree

## 13. Writes that preserve the same value

**Ambiguity:** A field assignment may write a value equal to the existing value, leaving the runtime predicate unchanged.

**Proposed resolution:** Any possible write to a contributing field invalidates the proof, even when the assigned value compares equal. The compiler tracks mutation events, not semantic equality or theorem-proved preservation.

agree

## 14. Mutation through aliases and calls

**Ambiguity:** A contributing field may be changed through an alias or inside another function without a visible assignment at the proof usage site.

**Proposed resolution:** Use First's ordinary ownership and alias information to find possible writes to recorded dependency paths. A call invalidates the proof when its mutability summary permits an overlapping write. If the compiler cannot exclude overlap, it invalidates conservatively.

agree

## 15. Mutation below a referenced field

**Ambiguity:** A proof may depend on a reference-valued field while another alias mutates the referenced object without replacing the field itself.

**Proposed resolution:** A dependency path includes the reachable field state actually read, not merely the top-level reference slot. Any possible mutation of that read state invalidates the proof. If alias analysis cannot represent the reachable path soundly, the issuer is outside the initial supported subset.

agree

## 16. Asynchronous suspension

**Ambiguity:** A contributing field may change while an issuing or consuming function is suspended at `await`.

**Proposed resolution:** Field proofs survive `await` only when ownership, alias, and execution-locality analysis proves that no dependency path can be mutated during suspension. Otherwise they are invalidated at the suspension point. Named-thread information may improve this analysis but does not replace alias checking.

agree

## 17. Worker boundaries

**Ambiguity:** Moving, sharing, cloning, or serializing field-proved values across Workers affects both identity and possible mutation.

**Proposed resolution:** Moving exclusive ownership preserves the proof and its dependency paths. Sharing preserves it only while no dependency path may be mutated. Compiler-known cloning follows the dedicated clone rule. Serialization and other new-value construction produce no field proofs.

agree

## 18. Compiler-known cloning

**Ambiguity:** Ordinary proofs copy unchanged, but a field proof's dependency paths are bound to fields of its carrier.

**Proposed resolution:** Compiler-known cloning copies the field proof and rebinds carrier-relative dependency paths to the corresponding fields of the clone. Dependencies rooted in external relationship objects remain bound to those same objects. Cloning is propagation, not issuance.

agree

## 19. Mutation during a bombing call

**Ambiguity:** A call may mutate a dependency and then return a bomb, after which recovery continues.

**Proposed resolution:** Once a contributing field may have been written, the proof is invalidated on every continuation, including bomb recovery. A bombing issuing path returns no new field proof.

agree

## 20. Automatic re-establishment

**Ambiguity:** The compiler may know the original issuer after invalidation, but it may lack required arguments or appropriate failure handling.

**Proposed resolution:** The compiler never re-runs an issuer automatically. Typist may link to the issuer and offer an AI-generated explicit revalidation edit when all arguments are available, but the programmer-visible call and bomb handling remain in source.

agree

## 21. External and ambient state

**Ambiguity:** A field proof could appear to depend on a database, clock, environment variable, or other state the compiler cannot observe for invalidation.

**Proposed resolution:** Field-proof issuers cannot read external or ambient state. Facts requiring external checks use ordinary durable proofs, operation-scoped runtime checks, or runtime capability objects. This restriction keeps field proofs honest and statically tractable.

agree

## 22. Dependency changes through prerequisite proofs

**Ambiguity:** A relationship parameter may require another field proof that later becomes invalid.

**Proposed resolution:** The outer field proof depends on the validity of every prerequisite field proof used during issuance. Invalidating a prerequisite also invalidates the outer proof, even if none of the outer issuer's directly read fields changed.

agree

## 23. Control-flow inference inside issuers

**Ambiguity:** Different validation branches may read different fields before reaching the same establishment expression.

**Proposed resolution:** The dependency set is the union of all field paths read on every feasible path that can reach successful issuance. Reads occurring only on paths that inevitably bomb do not contribute. The compiler performs ordinary intraprocedural reachability, not predicate theorem proving.

agree

## 24. Invalidation diagnostics

**Ambiguity:** A mutation can invalidate several field proofs indirectly through nested paths and aliases, and the required presentation is unsettled.

**Proposed resolution:** Typist marks the invalidating write, lists every proof it invalidates, shows the contributing dependency path, and links to the issuing site. The invalidation itself is explanatory; a later proof-dependent operation receives the prominent missing-evidence notice.

agree

## 25. Package metadata

**Ambiguity:** Separate compilation requires callers to know dependency and mutation summaries without seeing library bodies.

**Proposed resolution:** First package metadata retains field-proof declarations, inferred dependency schemas, issuer restrictions, exact proof contracts, and callable mutation summaries. Runtime artifacts erase all field-proof metadata.

agree

## 26. Initial implementation boundary

**Ambiguity:** The complete design could expand into arbitrary effect, alias, and dependency analysis.

**Proposed resolution:** The initial implementation supports direct stored-field reads, stable nested paths, compiler-visible pure helpers, whole-field collection dependencies, ordinary ownership aliases, and statically resolved calls. It rejects unsupported issuers with notices. Element-level collection precision, foreign summaries, arbitrary dynamic dispatch, user-asserted dependency lists, and whole-program inference are deferred.

agree

