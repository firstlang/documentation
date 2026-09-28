# Proofs

Proofs are durable, compile-time receipts that a named issuing function successfully established a fact.

A proof behaves like a wristband. Once issued, it remains attached to the same value for the lifetime of the compiler's knowledge of that value. Mutation does not remove it. Proofs are erased before runtime and add no storage, allocation, parameters, return values, or memory-management behavior.

Proofs guarantee provenance and flow:

- which nominal proof was issued
- which source site issued it
- which value carries it
- which runtime objects it relates to
- whether every consumer received the evidence it requires

Proofs do not ask the compiler to prove arbitrary predicates. They also do not guarantee that a mutable or externally controlled condition remains currently true.

---

# Declaring proofs

A proof is a nominal declaration:

```first
CredentialsAccepted is proof (
)
```

Proofs may be declared at root or space scope. They cannot be declared inside classes or functions.

The declaration syntax is:

```text
Name is proof (
	parameters
)

Name is federated proof (
	parameters
)
```

Two proof declarations are distinct even when they have identical names and parameters in different spaces:

```first
Internal (
	Approved is proof (
	)
)

External (
	Approved is proof (
	)
)
```

Ordinary qualification and import rules distinguish `Internal.Approved` from `External.Approved`. An alias preserves the identity and authority of the original proof rather than declaring another proof.

A proof's visibility governs its use in annotations and its issuance. A public API may expose only proofs and relationship-parameter types at least as visible as the API.

Every public proof must include documentation or an anchor stating the historical fact it certifies and any freshness limitation. Missing documentation produces a style notice. Typist and AI tooling may question misleading names such as `CurrentlyAuthorized`, but naming quality is not mechanically decidable and does not itself produce a compiler notice.

---

# Durable evidence

An ordinary proof records that issuance happened successfully.

```first
EmailVerified is proof (
)
```

If a value carries `EmailVerified`, the compiler guarantees that an authorized issuing site produced that evidence for the value. It does not independently verify what “email verified” means.

Mutation never removes an ordinary proof:

```first
user = verifyEmail(user)
// user carries EmailVerified

user.name = "Paul"
user.email = "another@example.com"

// user still carries EmailVerified
```

The proof remains truthful only as a historical statement: the issuing function verified the value at some earlier point. A proof intended to describe current mutable state must be named and documented carefully or modeled through a different mechanism.

Mutation-aware evidence is a separate future feature called a field proof. Field proofs are not defined by this document.

---

# Establishment authority

A non-federated proof has one issuing site.

```first
HtmlSafe is proof (
)

sanitize(value is string) is string with HtmlSafe (
	return escapeHtml(value) with HtmlSafe
)
```

The establishment expression `escapeHtml(value) with HtmlSafe` is the issuing site.

An issuing site is identified by stable token provenance. The expression may execute any number of times and issue evidence for any number of values. Repeated execution, inlining, generic monomorphization, and generated copies retaining one provenance origin do not create additional sites. Copying the establishment token creates another site.

If a non-federated proof has multiple issuing sites, every conflicting site receives a prominent notice, including the first. Evidence from every site remains usable so the program stays runnable.

Propagation is not issuance. Passing, returning, storing, aliasing, moving, or compiler-known cloning of existing evidence does not create another issuing site.

---

# Federated proofs

A federated proof permits multiple independent issuing sites:

```first
Imported is federated proof (
)
```

Federation changes only the uniqueness rule. Federated proofs have the same nominality, durability, relationships, erasure, and propagation behavior as other proofs.

Tooling retains the actual issuing provenance for each evidence flow.

---

# Where proofs may be issued

Establishment expressions may appear directly inside:

- named space functions
- named instance methods

They cannot appear inside:

- closures
- nested functions
- anonymous functions
- constructors
- properties
- other expression contexts

Issuing functions and methods are not first-class values. They cannot be passed, stored, returned, or invoked through callbacks. They must be called through First's statically resolved function or method dispatch.

A same-named method selected on another static type is a different function. If both methods establish one non-federated proof, they are conflicting issuing sites.

---

# Attaching proofs to values

In expression position, `with` issues and attaches one proof:

```first
safe = escaped with HtmlSafe
```

Relationship arguments follow the proof name:

```first
authorized = request with AuthorizedFor(user)
```

In type position, the same syntax requires evidence:

```first
render(html is string with HtmlSafe) (
	console.log(html)
)
```

Each establishment expression issues exactly one nominal proof. Several new proofs cannot be issued by one `with` expression.

A value may carry several proofs by composing evidence from separate issuing sites:

```first
verify(value is Document) is Document with Reviewed Approved 💣 (
	reviewed = review(value) 💣 throw
	return approve(reviewed) 💣 throw
)
```

The outer function propagates both proofs. It does not create issuing sites for them.

---

# Self-proving methods

A named instance method may issue a proof on its receiver and return the same object as `this with Proof`:

```first
User (
	constructor
	
	authenticate(password is string) is this with Authenticated 💣 (
		if passwordIsValid(password) (
			return this with Authenticated
		)
		
		throw "Authentication failed"
	)
)
```

First's static dispatch determines the selected method body and therefore the issuing site.

The runtime object is unchanged. `this with Authenticated` adds only compile-time evidence.

---

# Pure proofs

A proof application is pure when it has no runtime carrier. Purity belongs to the application, not the declaration.

A zero-parameter pure proof is issued using its bare name:

```first
Ticket is proof (
)

getTicket() is Ticket (
	return Ticket
)

enter(ticket is Ticket) (
	console.log("You're in")
)
```

Usage:

```first
ticket = getTicket()
enter(ticket)
```

After erasure, the runtime call is equivalent to:

```first
enter()
```

`Ticket()` is invalid. A bare proof name is the zero-parameter pure-proof expression.

All valid instances of the same zero-parameter nominal pure proof satisfy the same requirement. The compiler retains issuing provenance for auditing, but First source cannot inspect or compare individual proof instances.

---

# Relationship proofs

Proof parameters relate evidence to particular runtime objects:

```first
AuthorizedFor is proof (
	user is User
	resource is Resource
)
```

Attached use:

```first
request with AuthorizedFor(user, resource)
```

Pure use:

```first
authorization = AuthorizedFor(user, resource)
```

The parameterized pure form uses parentheses because it supplies relationships. It does not construct a runtime object. Empty call syntax remains invalid.

A relationship parameter may require:

- a reference value with compiler-trackable runtime identity
- a primitive value that already carries another proof

It cannot accept an unproved integer, boolean, string, literal, type argument, function, proof instance, temporary expression, or arbitrary compile-time value.

Proof parameters carry no inspectable data. For a reference value, the compiler records opaque symbolic identity:

```text
AuthorizedFor(User#17, Resource#42)
```

For a proved primitive value, the relationship follows the compiler-tracked proved value flow rather than exposing or comparing its underlying scalar data.

Relationship arguments are positional, use ordinary assignment compatibility, and require exact arity. They cannot be optional, defaulted, or variadic.

A relationship argument must be a side-effect-free access path to an existing eligible value. Calls, construction, assignment, `await`, and other effectful expressions cannot appear as relationship arguments.

Aliases of the same runtime object satisfy the same relationship. Rebinding a source name does not retarget existing evidence:

```first
owner = alice
document = establishOwner(document, owner)

owner = bob

// The evidence still relates document to alice's object identity.
```

A proof parameter may itself require evidence:

```first
AuthorizedFor is proof (
	user is User with Authenticated
	resource is Resource
)
```

The prerequisite is checked when `AuthorizedFor` is issued. The outer proof records the related value's opaque identity, not inspectable proof data.

Because ordinary proofs are durable, later mutation of the carrier or any related object does not invalidate the relationship.

---

# Explicit function contracts

A function or method returning evidence must declare its complete proof-bearing return type. Proof-bearing results are never inferred.

Every successful return must carry exactly the declared proof set with identical relationship arguments:

```first
authorize
	(request is Request, user is User) 
	is Request with AuthorizedFor(user) 💣 (
	
	if policyAllows(request, user) (
		return request with AuthorizedFor(user)
	)
	
	throw "Not authorized"
)
```

A return path cannot silently add, omit, join, narrow, or discard proofs. A path unable to satisfy the contract must bomb or return an explicitly declared nominal alternative.

A proof-bearing return annotation is a requirement on the implementation. It does not itself issue evidence.

---

# Parameters and typed views

A value may satisfy a parameter requiring any subset of its proofs:

```first
inspect(value is Document with Reviewed) (
)

document is Document with Reviewed Approved
inspect(document)
```

Inside `inspect`, the parameter exposes only `Reviewed`. The function is not implicitly generic over the additional `Approved` proof.

The caller's original binding retains its stronger evidence. A returned binding, field, or collection access exposes exactly the proof set declared by its type, even when another binding for the same runtime object carries more evidence.

Initial First does not provide proof-row polymorphism.

There is no explicit proof-removal operation. A narrower typed view can hide evidence, but it does not remove evidence known through another binding.

---

# Control-flow analysis

Proof availability is path-sensitive at the usage site:

```first
if condition (
	value = establishApproved(value)
	consumeApproved(value)
)

// Approved is not available on every path here.
consumeApproved(value) // notice
```

A proof-dependent operation is valid only when every path reaching it carries the required nominal proof with identical relationships.

The compiler does not create or expose a reduced merged proof type.

Generics require no special proof semantics. Ordinary monomorphized types are checked normally, while proof evidence flows as separate compiler metadata.

---

# Mutation, asynchronous code, and Workers

Mutation has no effect on ordinary proofs:

```first
user = authenticate(user)
user.name = "Paul"

// Authenticated remains attached.
```

The same rule applies to:

- direct field mutation
- mutation through aliases
- calls that mutate a carrier or related object
- `await`
- Worker transfer or shared Worker access
- foreign mutation
- externally changed state

No freezing, dependency tracking, effect analysis, or automatic revalidation is required for ordinary proofs.

This durability is a semantic limitation as well as a convenience. The proof means only that its issuer succeeded historically.

---

# Freshness and external state

An erased proof cannot observe revocation, time, databases, networks, or policy changes.

```first
authorize(user is User) is User with AuthorizationChecked 💣 (
	if database.isAuthorized(user.id) (
		return user with AuthorizationChecked
	)
	
	throw
)
```

`AuthorizationChecked` means the database check succeeded when `authorize` returned. It does not mean the database still authorizes the user.

Facts requiring current external truth must use one or more of:

- a fresh runtime check
- operation-scoped evidence
- an immutable request snapshot
- an expiring or revocable runtime capability
- another runtime protocol

Proof documentation must make this boundary clear.

---

# Propagation and new values

The following preserve evidence known through the binding:

- assignment to an equally proved binding
- aliasing
- borrowing
- moving
- returning the same value under an exact proof-bearing contract
- mutation of the same value

A typed parameter or storage location exposes only its declared proofs without removing stronger evidence known through another binding.

An operation that creates a distinct value does not inherit proofs unless it explicitly obtains evidence from an authorized issuer.

The one exception is the compiler-known clone operation.

---

# Compiler-known cloning

The compiler-known clone operation creates a new runtime identity and copies the complete proof set unchanged:

```first
copy = Object.clone(original)
```

If `original` carries:

```text
Reviewed
Approved
OwnedBy(alice)
```

then `copy` carries the same three proofs. Relationships continue to name the same related objects, including `alice`. Nothing is retargeted or reinterpreted.

Cloning propagates evidence and creates no issuing site.

User-defined functions named `clone`, copy constructors, serialization, conversion, and other clone-like operations do not receive this behavior.

---

# Fields and collections

Fields and collection elements may require proofs:

Every inserted value must satisfy the declared proof requirements. Retrieval exposes exactly the proof set declared by the storage type.

A homogeneous collection cannot expose different proof sets for different elements. Use explicit nominal alternatives when elements require heterogeneous proved states.

Evidence on an object does not automatically attach to its members, and evidence on a member does not attach to its containing object. Any such relationship requires an explicitly issued relationship proof.

---

# Runtime erasure

Proofs never exist at runtime.

They introduce:

- no allocation
- no storage
- no runtime parameters
- no runtime return values
- no generated proof objects
- no reference counting
- no runtime reflection surface

Proofs cannot participate in:

- attestations
- runtime type tests
- branching
- equality
- enumeration
- inspection
- runtime reflection

The compiler removes proof-only values and parameters after semantic analysis.

---

# Bombs and recovery

An issuing function may bomb when validation fails:

```first
verify(value is Document) is Document with Reviewed 💣 (
	if reviewPasses(value) (
		return value with Reviewed
	)
	
	throw "Review failed"
)
```

A bombing path issues no returned evidence. Every successful path must satisfy the exact declared proof contract.

Missing, inaccessible, or invalid evidence produces a prominent notice. The protected operation is cooked:

- its receiver is not evaluated
- its arguments are not evaluated
- no runtime call is emitted
- ordinary cooked-expression recovery continues execution

Recovery never bypasses a proof requirement.

An invalid attachment evaluates and returns its underlying runtime value but attaches no evidence. An invalid pure-proof expression produces an unknowable compile-time placeholder that satisfies no proof requirement. Establishment in a forbidden context does not count as an issuing site.

---

# Foreign code

Foreign code is a proof dead-zone.

Raw foreign declarations cannot mention proofs because their runtime ABI has no proof representation. A named First function or method may wrap foreign code, require proofs before calling it, and issue proofs on its result under the ordinary authority rules.

Foreign mutation does not invalidate durable proofs.

A compiled First package retains its proof information. A Rust crate or other foreign package has no proof metadata of its own.

---

# Package metadata and tooling

Separately compiled First package interfaces retain:

- nominal proof identity
- relationship parameter types
- federation
- proof requirements
- exact returned proof sets
- issuing-site provenance

Executable artifacts erase proofs.

Read-only compiler and Typist APIs may inspect proof declarations, requirements, relationships, flows, and provenance for checking and presentation. Ordinary First runtime code cannot access this metadata.

Typist should:

- display attached and required proofs
- navigate to declarations and issuing sites
- show related runtime objects
- explain missing evidence
- show public proof documentation and freshness limitations
- expose proof flow through prismatic audit views

---

# Relationship to types

Proofs extend ordinary types without changing runtime representation:

```first
string
string with HtmlSafe
Document with Reviewed Approved
Request with AuthorizedFor(user)
```

The ordinary type describes runtime shape and behavior. Proofs describe durable evidence known by the compiler.

Proofs are nominal and cannot be manufactured through casting, attestation, structural compatibility, or return annotations.

---

# Relationship to `declare`

`declare` applies compiler-defined invariants and policies to lexical scopes and declarations.

Proofs are user-defined nominal evidence carried through value flow. The mechanisms remain separate.

---

# Appropriate uses

Ordinary proofs work best for historical, immutable, monotonic, or operation-scoped facts:

- credentials were accepted
- a string was escaped
- an audit completed
- consent was recorded
- data came from a particular source
- a migration ran
- a protocol stage completed
- an immutable request was authorized
- a buffer was initialized when initialization is irreversible

They are not sufficient by themselves for continuing claims about revocable external state or mutable current-state invariants.

---

# Summary

An ordinary First proof is a durable, nominal, compiler-tracked receipt.

It:

- is issued by an auditable named function or method
- has one issuing site unless federated
- may be attached or pure
- may relate opaque runtime identities
- survives every mutation
- propagates through existing-value flow
- copies through the compiler-known clone operation
- requires explicit and exact function contracts
- is uninspectable and fully erased at runtime
- proves historical issuance rather than current mutable truth

Field proofs, which conditionally survive mutation according to field dependencies, are a separate feature and are not defined here.
