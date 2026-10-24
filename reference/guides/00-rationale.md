# Rationale and Edge Cases

Status: Supporting guide
Normative source: [../core/00-rules.md](../core/00-rules.md)

This guide explains why the rules exist and how to resolve cases that do not fit a short example. It is not normative.

## 1. Why four verbs

`insert`, `find`, `update`, and `remove` describe persistence operations without borrowing UI, HTTP, or collection vocabulary. `create` usually means "construct a domain object"; `save` hides whether the operation inserts or updates; `get` hides whether absence is normal or an error; `list` and `getAll` hide cardinality and scope.

The vocabulary is intentionally narrow. Distinct lifecycle operations such as `archive`, `restore`, `publish`, and `purge` may keep their own verbs because they express domain transitions rather than CRUD synonyms.

## 2. Why `By` conditions

A positional call such as `findOne(id)` does not say what `id` means. `findOneById` does. The `By` segment also makes multi-condition methods readable and prevents storage details from leaking into the contract.

`Or` is excluded from names because boolean combinations become unreadable and easy to misread. A specification or criteria object is preferable: `findManyBySpecification(specification)`.

## 3. Why no repeated aggregate noun

The enclosing type already establishes the aggregate. `IUserRepository.findOneById` reads as "find one user by id" without repeating `User`. Repetition increases noise and makes cross-aggregate conditions harder to distinguish from the owning aggregate.

A condition on another aggregate is different: `IOrderRepository.findManyByCustomerId(customerId)` is correct because `Customer` is the condition's aggregate, not the repository's aggregate.

## 4. Why `find` over `get`

Absence is usually a normal query outcome. Encoding it as an exception forces every caller into try/catch and makes control flow harder to read. `find` keeps absence in the return type.

`get` remains useful when the surrounding use case genuinely requires the record: a profile page, a command that cannot proceed, or a referential invariant. In those cases, absence is an error and a typed not-found exception or error result is correct.

`tryGet` sits between them. It is appropriate when the caller must branch on absence without exceptions but the API does not want a nullable value. To be meaningful, its result must be distinct from `find`; returning the same optional value adds no information.

## 5. Why the identity-bearing update is allowed

An aggregate root carries its identity. `updateOne(user)` is therefore bounded even without a `By` segment: the parameter itself identifies the target. The same is true for `updateOne(id, patch)`.

`updateOneById` is still preferred because the condition is visible at the call site and in logs. The exception is deliberately narrow: it applies only to the single-update form. `removeOne`, `updateMany`, and `removeMany` must always state a condition, because there is no single identity parameter that bounds them.

The implementation must reject a missing, null, empty, or invalid identity before issuing the statement. A default identifier or an empty struct is not a valid target.

## 6. Why `findMany()` means all

A parameterless plural read is useful and predictable: it returns every record in the repository's declared visibility scope. In a tenant-scoped repository, that means every record visible to the tenant, not every row in the database.

The scope must be documented on the repository or its constructor contract. Hidden filters are dangerous because a caller cannot tell why a record is missing. If the repository is policy-scoped, the policy belongs in the contract documentation and tests.

## 7. Why `Many` over `All`

`Many` describes cardinality; `All` describes a set that may be unbounded and often implies a full table scan. `Many` composes with `One` and makes the single/plural distinction symmetric. `findMany()` is also easier to pair with `findManyBy...` than `findAll()` with `findAllBy...`.

Some projects already use `All` in a public API. Replacing the vocabulary is acceptable as an adapted profile if cardinality remains distinguishable and the parameterless plural read still returns the full scope. It is not a core waiver. The decision register must record the complete mapping, including updates and removals.

## 8. Why inserts are retained in an All-based profile

`insertOne` and `insertMany` are part of the R1 insert contract, not the read/update/removal scope vocabulary. Renaming them to `insert` and `insertAll` would add no clarity and would break the established insert forms. An All-based profile therefore replaces `One`/`Many` only where it affects query scope.

## 9. Adaptation and waivers

A configurable choice changes the surface while preserving every core rule. Examples: `Single`/`Multiple`, `All`-based scope, an async suffix, a repository `get` opt-in, or returning the updated entity instead of a boolean.

A waiver gives up a core rule. Examples: using `create` as the insert verb, using `get` as the default read, or repeating the aggregate noun in every member. A waiver requires a decision, a partial profile, and an explicit statement that the project is not compliant.

Do not use "adaptation" to hide a waiver. If the change removes a guarantee that a caller depends on, it is a waiver.

## 10. Edge cases

- **Singleton aggregate.** `findOne()` without a condition is permitted only when the contract guarantees one instance and documents it. Otherwise use `findMany()` with a limit or a condition.
- **Projection repository.** Name it for the projection: `IActiveUserSummaryRepository`. Do not return a different aggregate from a repository whose type name establishes another aggregate.
- **Generic repository.** `Repository<T>` establishes no noun, so members use `findOneById`, `findMany`, and `findManyBySpecification`. Do not invent `findOneByUserId` for a generic type.
- **Cross-aggregate read.** Prefer a dedicated repository or read model. If unavoidable, record it as an extension and do not repeat the enclosing noun.
- **Framework methods.** Framework lifecycle methods are outside the naming rules. Document them as framework surface.
- **Generated clients.** Change the generator or wrap the generated surface. Do not hand-edit generated code unless the project owns it.
- **Scalar helpers.** `count`, `countBy...`, and `existsBy...` are optional. They return scalars, so they do not use `One`/`Many`/`All` tokens.
- **Localized identifiers.** A domain term in another language may be used when it is the project's ubiquitous language. Record abbreviations and translations in the glossary or decision register.

## 11. Common objections

**"Our public API already uses `getAll`."**  
Keep it as a compatibility alias for the approved window, then migrate. Or record an All-based adaptation if the public vocabulary cannot change. Do not silently mix both.

**"`get` is shorter and everyone understands it."**  
Short names are not worth ambiguous absence semantics. Use `get` only where absence is an error, and make that contract explicit.

**"`updateOne(user)` has no condition, so it is unsafe."**  
It is bounded by the aggregate's id. The repository must validate the id. If the parameter could lack an id, use `updateOneById(id, patch)`.

**"We need a bulk update with no condition for a migration."**  
A one-off migration belongs in a migration script, not a repository contract. If a repository method must do it, require an explicit condition or a recorded waiver.

**"The rule set is too strict for a small service."**  
Use the compact mode and the adapted profile. The core invariants still apply; the vocabulary can change.

## 12. Review heuristics

- If a name needs a comment to explain its cardinality, the name is wrong.
- If a caller cannot tell whether absence is normal, the return type is wrong.
- If a mutation can run without a condition or identity, it is unsafe.
- If a condition contains a storage column, it is leaking persistence.
- If two methods differ only by a synonym, one is redundant.
- If an adaptation is not in the register, it is drift.
