# Naming Grammar

Status: Normative application guide
Source: [00-rules.md](00-rules.md)

This file defines how to read and construct repository method names. The rule catalog is normative; this guide applies it.

## 1. Segment order

```text
MethodName      := Verb Scope? Condition? TransportSuffix?
Verb            := Insert | Find | Update | Remove
Scope           := One | Many | AdoptedScope
Condition       := By ConditionTerm (And ConditionTerm)*
TransportSuffix := a framework-mandated suffix such as Async
```

`AdoptedScope` is the R8-approved cardinality vocabulary, such as an implicit single form plus `All`. The semantic token sequence is normative. Casing follows the host language.

Examples:

```text
insertOne
insertMany
findOneById
findManyByStatus
findManyByStatusAndTenantId
updateOneById
updateManyByStatus
removeOneById
removeManyByStatus
```

## 2. Segment rules

| Segment | Required? | Rule |
| --- | --- | --- |
| Verb | Required | Exactly one of `insert`, `find`, `update`, `remove`, or a documented lifecycle verb. |
| Scope | Required for canonical CRUD | `One` for at most one target; `Many` for zero or more; or the R8-approved vocabulary (for example, implicit single plus `All`). |
| Condition | Required for explicit conditions | `By<Condition>` after scope; parameters in name order. |
| Transport suffix | Optional | Framework-mandated, final, singular, outside the semantic name. |

Scope is mandatory for canonical CRUD even though the grammar marks it optional. In the default profile, `findMany()` is the only parameterless plural read and `findOne()` is permitted only for a documented singleton aggregate. In an All-based profile, `findAll()` is the parameterless plural read and `findById`/`findBy<UniqueCondition>` are the single forms. `findByStatus` is ambiguous and prohibited in every profile.

The insert forms `insertOne` and `insertMany` retain their suffix in every profile, including an All-based profile.

## 3. Conditions

Every explicit selecting condition uses `By`:

```text
findOneById
findOneByEmail
findManyByStatus
findManyByStatusAndTenantId
findManyByIdIn
findManyByCreatedAtBetween
findManyByDeletedAtIsNull
findManyBySpecification
```

Rules:

- Parameters follow the condition order.
- Multiple predicates join with `And`.
- `Or` MUST NOT appear in a name; use a specification or criteria object.
- Do not encode database column names, positional indexes, or storage details.
- `In` means set membership; `Between` means a range; `Not` means negation; `IsNull` and `IsNotNull` mean nullability.
- A complex boolean query uses one segment such as `findManyBySpecification`.

## 4. Cardinality and scope

| Token | Meaning | Typical result |
| --- | --- | --- |
| `One` | At most one target | Record, absence value, boolean, or single-record count |
| `Many` | Zero or more targets | Non-null collection or affected count |

`Many` is preferred over `All`. `All` is prohibited in the default profile. An All-based profile may replace the `One`/`Many` vocabulary only under a recorded decision; it must preserve distinguishable cardinality and a parameterless plural read that returns the full declared scope.

## 5. Identity-bearing single update

A single-aggregate update may omit `By` when a required parameter carries the aggregate root's unique identifier.

```text
updateOne(user)              // valid: user carries its id
updateOne(id, patch)         // valid: id is the required identity
updateOneById(id, patch)     // preferred: condition is visible
updateOne(patch)             // invalid: no identity
updateOne()                  // invalid: no target
```

The implementation MUST reject a missing, null, empty, or invalid identifier before issuing the statement. The exception applies only to the single-update form. `removeOne`, `updateMany`, and `removeMany` always require a `By` condition and a runtime guard.

In an All-based profile, `updateById` is preferred; `update(user)` is permitted only when the parameter carries the aggregate id. `update(patch)` and `update()` are invalid.

## 6. Transport suffixes and casing

- `Async` is a suffix, not a semantic segment: `FindOneByIdAsync`, not `FindByIdAsyncOne`.
- C#: PascalCase; `Async` when the platform convention requires it.
- TypeScript/JavaScript: camelCase.
- Java/Kotlin: camelCase.
- Python: snake_case.
- Go: PascalCase for exported members; `ID` is an accepted acronym style.
- Rust: snake_case.

The project chooses one casing convention per language and applies it consistently. Acronym casing is configurable under R8.

## 7. Entity-noun suppression

The enclosing type establishes the aggregate noun.

```text
IUserRepository.findOneById(id)             correct
IUserRepository.findOneUserById(id)         incorrect
IUserRepository.findManyByStatus(status)    correct
IUserRepository.findManyUsersByStatus(...)  incorrect
```

A condition may contain another aggregate's noun: `IOrderRepository.findManyByCustomerId(customerId)` is correct. A cross-aggregate read should use a dedicated repository or a read model whose name establishes the result.

## 8. Edge cases

- **Singleton aggregate.** `findOne()` without a condition is permitted only when the contract guarantees exactly one instance and documents it.
- **Generic repository.** `Repository<T>` establishes no aggregate noun; members still use `findOneById`, `findMany`, `findManyBySpecification`, `insertOne`, `updateOneById`, `removeOneById`.
- **Projection.** Name the projection repository for its result, such as `IActiveUserSummaryRepository.findManyByTenantId(tenantId)`.
- **Lifecycle operation.** `archiveOneById`, `restoreOneById`, `publishOneById`, and `purgeOneById` are permitted when they are genuinely distinct from CRUD and documented.
- **Composite operation.** `upsertOneById` requires a decision register entry and must not be a convenience alias for insert or update.
- **Framework method.** A framework lifecycle method is documented as framework surface and is not renamed.
- **Scalar helper.** `count()`, `countByStatus(status)`, and `existsById(id)` are optional. They do not use `One`/`Many`/`All` tokens because their result is scalar. They never replace a CRUD verb.

## 9. Anti-patterns

| Incorrect | Correct | Rule |
| --- | --- | --- |
| `createUser(user)` | `insertOne(user)` | R1 |
| `save(user)` | `insertOne(user)` or `updateOneById(id, patch)` | R1 |
| `getUserById(id)` | `findOneById(id)` or service `getOneById(id)` | R1, R4 |
| `listUsers()` | `findMany()` | R1, R5 |
| `getAllUsers()` | `findMany()` | R1, R5 |
| `deleteUserById(id)` | `removeOneById(id)` | R1 |
| `findById(id)` in the default profile | `findOneById(id)` | R5, R6 |
| `findByStatus(status)` | `findManyByStatus(status)` | R2, R5 |
| `findManyUsers()` | `findMany()` | R3, R5 |
| `findOneUserById(id)` | `findOneById(id)` | R3 |
| `updateOne(patch)` | `updateOneById(id, patch)` or `updateOne(user)` | R2, R5 |
| `updateMany()` | `updateManyByStatus(status, patch)` | R5 |
| `removeOne(id)` | `removeOneById(id)` | R2, R5 |
| `removeMany()` | `removeManyByStatus(status)` | R5 |
| `findManyByStatusOrTenantId(...)` | `findManyBySpecification(specification)` | R2, R6 |
| `countUsers()` | `count()` or `countByStatus(status)` | R1, R8 |

## 10. Review algorithm

Apply these points in order:

1. Confirm the type is in scope and identify the aggregate noun.
2. Strip the transport suffix; confirm it is final and outside the semantic name.
3. Check the verb: exactly one of `insert`, `find`, `update`, `remove`, or a documented lifecycle/composite verb.
4. Check the scope: `One` or `Many` for canonical CRUD, or the adopted R8 vocabulary, matching the intended cardinality; parameterless forms are limited to `findMany()` (or the adopted plural form) and a documented singleton `findOne()`.
5. Check the condition: explicit conditions use `By`; a single aggregate update may rely on an identity-bearing parameter; no condition on an insert.
6. Check the aggregate noun: it must not repeat the enclosing type's noun.
7. Check parameter order and domain predicates.
8. Check the return contract and absence semantics.
9. Check the wording against the project decision register.
10. Record a pass or a finding with a file and symbol.
