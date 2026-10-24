# Core Rules

Status: Normative
Version: rule catalog 1.0
Managed block version: v1

This file is the source of truth for the CRUD Scope Standardizer. Other documents explain, apply, or exemplify these rules; they do not override them. When a conflict exists, this file prevails.

## 1. Scope and precedence

The rules apply to repository interfaces, traits, protocols, abstract classes, and the public surface of their implementations; to service, gateway, and port interfaces whose type name identifies the aggregate; and to generated clients only when the project configures or wraps the generator.

The rules do not apply to ORM entities, database schema objects, migration files, framework-fixed lifecycle methods, private helpers outside a contract, or transport route names, JSON keys, and payload fields, unless the project explicitly adopts the same vocabulary for them.

Precedence, highest first:

1. these core rules;
2. the project decision register for configurable choices;
3. the default profile;
4. language and framework conventions for casing, suffixes, and transport types;
5. local formatting preferences.

A lower item MUST NOT override a higher item. If a conflict cannot be resolved by this order, stop and report it.

## 2. Terms

- **Aggregate.** The domain object owned by a repository contract. In `IUserRepository`, the aggregate is `User`.
- **Aggregate noun.** The noun in the enclosing type name that identifies the aggregate. Members do not repeat it.
- **Condition.** A predicate that selects records to return or affect. Explicit conditions use `By`.
- **Scope.** The cardinality of a CRUD operation: `One` for at most one target, `Many` for zero or more.
- **Absence.** A normal query outcome with no matching record. Absence is distinct from failure.
- **Decision register.** The durable record of configurable adaptations and core-rule waivers.
- **Managed block.** The single region of an instruction file that emission owns.

## R1. CRUD verbs

Repository create, read, update, and delete operations MUST use exactly `insert`, `find`, `update`, and `remove`.

Canonical forms: `insertOne`, `insertMany`, `findOneBy...`, `findMany`, `findManyBy...`, `updateOneBy...` (preferred), `updateOne` (identity-bearing), `updateManyBy...`, `removeOneBy...`, `removeManyBy...`.

Correct: `insertOne(user)`, `findOneById(id)`, `updateOneById(id, patch)`, `removeOneById(id)`.

Incorrect: `createUser`, `save`, `getUserById`, `listUsers`, `deleteUserById`.

Distinct lifecycle operations may keep their own verbs, such as `archiveOneById` or `restoreOneById`, when documented and not CRUD synonyms. A composite verb such as `upsertOneById` requires a decision register entry. Framework-mandated methods are framework surface and are not renamed.

## R2. Conditions

Every repository method that selects records by an explicit condition MUST encode it as `By<Condition>` after the scope segment and before any transport suffix. The condition uses a domain predicate, matches the order of semantic parameters, and joins multiple predicates with `And`.

Use `In` for set membership, `Between` for ranges, `Not` for negation, and `IsNull` or `IsNotNull` for nullability.

Correct: `findOneById(id)`, `findManyByStatusAndTenantId(status, tenantId)`, `findManyByIdIn(ids)`, `removeOneByEmail(email)`.

Incorrect: `findOne(id)`, `findMany(status)`, `findManyByStatusOrTenantId(...)`, `findManyByColumnC3(value)`.

The aggregate's intrinsic identity is not an explicit condition. A single-aggregate `updateOne` MAY omit `By` when a required parameter carries the aggregate root's unique identifier. `updateOneById` remains preferred because it makes the condition visible. All non-identity conditions MUST use `By`.

Correct: `updateOne(user)`, `updateOne(id, patch)`, `updateOneById(id, patch)`.

Incorrect: `updateOne(patch)`, `updateOne()`.

`removeOne`, `updateMany`, and `removeMany` MUST NOT omit a condition. A singleton aggregate may expose `findOne()` only when the contract guarantees exactly one instance and documents that fact. Complex boolean logic uses one condition segment, such as `findManyBySpecification(specification)`; `Or` MUST NOT appear in a method name.

## R3. Aggregate noun

A member name MUST NOT repeat the aggregate noun established by the enclosing type name.

Correct: `IUserRepository.findOneById(id)`, `IOrderRepository.findManyByCustomerId(customerId)`.

Incorrect: `IUserRepository.findOneUserById(id)`, `IUserRepository.findManyUsersByStatus(status)`.

A condition may contain another aggregate's noun when it refers to that aggregate. A cross-aggregate read should use a dedicated repository or a read model whose name establishes the result.

## R4. Read semantics

The read families MUST have distinct contracts. Priority is `find`, then `tryGet`, then `get`.

- `findOneBy...` returns the record or an absence value; absence is not an exception.
- `findMany` and `findManyBy...` return a non-null collection, empty when there are no matches.
- `tryGetOneBy...` returns an explicit success-or-absence result and does not throw for absence.
- `getOneBy...` returns the record or raises/returns a typed error; it never returns an absence value.
- `getManyBy...` is used only where the contract requires a non-empty result; otherwise use `findManyBy...`.

The default repository read surface exposes `find` only. `get` and `tryGet` belong at an application, service, or boundary layer, unless the project records a repository opt-in under R8. A `findOne` or `getOne` condition MUST guarantee uniqueness; a non-unique condition uses `findManyBy...` with an explicit limit and documented order.

## R5. Cardinality and safety

Every CRUD method MUST make cardinality distinguishable. The default vocabulary is `One` for at most one target and `Many` for zero or more. `Many` is preferred over `All`; `All` is prohibited in the default profile.

`findMany()` with no semantic parameters returns all records in the repository's declared visibility scope, with documented default ordering. `findManyBy<Condition>` returns all matching records in that scope. Presentation and infrastructure parameters do not change the meaning of a parameterless operation.

Correct: `findMany()`, `findManyByStatus(status)`, `updateOne(user)`, `updateOneById(id, patch)`, `removeManyByIdIn(ids)`.

Incorrect: `findAll()`, `findByStatus(status)`, `findManyUsers()`, `updateOne(patch)`, `updateOne()`, `updateMany()`, `removeMany()`.

`updateManyBy...` and `removeManyBy...` MUST reject an empty or missing condition at runtime. An empty identifier collection in `removeManyByIdIn(ids)` is a no-op and is never interpreted as removing all records. The identity exception for `updateOne` does not relax bulk safety.

In a multi-tenant or policy-scoped repository, "all" means all records visible to the declared scope. The scope MUST be documented on the repository or its constructor contract; hidden filters are prohibited.

A project MAY replace the `One`/`Many` vocabulary under R8, including an All-based profile, provided cardinality remains distinguishable and a parameterless plural read returns the full declared scope. The replacement applies to read, update, and removal scope; `insertOne` and `insertMany` are retained from R1. Replacing the vocabulary is a configurable adaptation, not a core waiver.

## R6. Grammar

Canonical method names follow this grammar, with fixed segment order:

```text
MethodName      := Verb Scope? Condition? TransportSuffix?
Verb            := Insert | Find | Update | Remove
Scope           := One | Many | AdoptedScope
Condition       := By ConditionTerm (And ConditionTerm)*
TransportSuffix := a framework-mandated suffix such as Async
```

`AdoptedScope` is the cardinality vocabulary approved under R8, such as an implicit single form plus `All`. Its position and distinguishability are mandatory; the tokens are a recorded configurable choice. `insertOne` and `insertMany` retain their forms in every profile.

Casing follows the host language: `FindOneByIdAsync` in C#, `findOneById` in TypeScript, `find_one_by_id` in Python, `FindOneByID` in Go. The token sequence is normative; acronym casing is not.

A method name MUST NOT contain the repository type name or the word `Repository`, place `Async` or another transport marker inside the semantic segments, or abbreviate domain terms unless the abbreviation is recorded. `Or` MUST NOT appear in a method name. Ordering and pagination use presentation parameters; an ordering suffix such as `findManyByStatusOrderByCreatedAtDesc` requires an R8 entry.

A bare `updateOne` is permitted only when a required parameter carries the aggregate root's unique identifier. A bare `findOne` is permitted only for a singleton aggregate. A bare `removeOne` is prohibited.

## R7. Return and error contracts

Return and error types MUST make the selected semantics observable and MUST be consistent within a contract.

- `findOneBy...` returns the record or an absence value.
- `findMany` and `findManyBy...` return a non-null collection.
- `getOneBy...` returns the record or a typed error; it never returns an absence value.
- `tryGetOneBy...` returns an explicit result and does not throw for absence.
- `insertOne` returns the inserted record, its identifier, or both under a documented policy.
- `insertMany` returns the inserted records, their identifiers, or an affected count under the same policy.
- `updateOne`, `updateOneBy...`, and `removeOneBy...` return an affected count or a boolean under a documented policy.
- `updateOne` and `updateOneBy...` MAY return the updated record instead; `removeOneBy...` MUST NOT return a record.
- `updateManyBy...` and `removeManyBy...` return an affected count or an equivalent bulk result.

Do not swallow infrastructure errors. Translate them at the appropriate boundary while preserving the distinction between absence and failure.

## R8. Adaptation and the decision register

A deviation from the default profile MUST be recorded as a decision. Core rules that MUST NOT be waived while claiming compliance are R1, R2, R3, R4, R5, R6, R7, and R10. R6 grammar is mandatory; the scope-token vocabulary and surface conventions such as casing and transport suffixes may vary when recorded.

Configurable choices include cardinality tokens, the `All` scope, repository `get`/`tryGet` placement, transport suffixes, acronym casing, mutation return policy, scalar helpers, ordering in names, compatibility strategy, emission target, and enforcement scope.

Each register entry records: ID, Rule, Default, Decision, Rationale, Scope, Consequence, Approver, Date, Review.

A core-rule waiver places the project in the partial profile. Replacing the cardinality vocabulary is an adaptation, not a waiver, provided R5's invariant remains satisfied. An unrecorded deviation is a defect to report, not to implement. A register entry that conflicts with a core rule is invalid.

A strict project with no adaptation or waiver MAY omit the register, provided the emitted block states that no decisions exist. A register is required as soon as a deviation exists.

## R9. Emission

Emission writes the adopted contract into a target instruction file. It requires an explicit instruction and a confirmed target.

Target selection order: an explicit path; `AGENTS.md` at the project root; `CLAUDE.md` at the project root; otherwise ask and do not guess. If both files exist and no target is chosen, ask. If both are chosen, both blocks must carry the same profile and rule set.

The emitted content is one managed region delimited by the opening and closing marker lines of the block. Merge procedure:

1. Read the target.
2. If exactly one managed region exists, replace its content and preserve everything outside it.
3. If no managed region exists, append the block after a blank line.
4. If multiple regions exist, stop and ask before consolidating.
5. Preserve line endings.
6. Do not rewrite, reorder, or reformat unrelated content.
7. Re-read and verify exactly one region, the expected profile, and register alignment.

Emission is idempotent and non-destructive. It does not delete user-authored rules; it reports conflicts. The block must state the adopted profile and compliance status, the applicable rules in the project's vocabulary, the confirmation gate, a catalog pointer, the register location, and the enforcement mechanism.

## R10. Migration confirmation gate

A migration MUST proceed through five phases:

| Phase | Mode | Allowed actions | Exit condition |
| --- | --- | --- | --- |
| A. Audit | Read-only | Inspect architecture, repositories, call sites, tests, constraints | Audit report produced |
| B. Plan | Read-only | Produce feasibility, benefits, drawbacks, breaking changes, steps, target architecture, risks, rollback | Plan presented |
| C. Confirmation | No writes | Ask for explicit approval, answer questions, revise | Approval recorded |
| D. Execution | Writes allowed | Apply the approved plan in reviewed increments | Each increment verified |
| E. Validation | Approved fixes and reports | Run checks, produce a validation report, close or reopen findings | Acceptance criteria met |

No source modification may occur before explicit confirmation of the plan. Silence, a request for more detail, or an ambiguous comment is not approval. A material change to scope, target architecture, or breaking-change classification requires a new confirmation.

## 3. Profiles and compliance

| Level | Definition | Permitted label |
| --- | --- | --- |
| Strict | All core rules satisfied, default profile, no adaptations or waivers | Compliant with CRUD Scope Standardizer, strict profile |
| Adapted | All core rules satisfied, configurable choices recorded | Compliant with CRUD Scope Standardizer, adapted profile |
| Partial | One or more core rules waived or violated, with a recorded waiver | Partially aligned; not compliant |
| Non-compliant | Core rules violated without a recorded waiver, or ambiguous naming remains | Not compliant |
| Audit-only | No migration or emission performed | Audited only; compliance not asserted |

An audit reports a candidate level. The supported label requires validation evidence. A project MUST NOT be described as compliant if a core rule is waived, even when the waiver is approved.

## 4. Change control

A change to a core rule requires a major catalog version. A change to a configurable choice, example, or template requires a minor version. The managed block marker version tracks the catalog major version. Deprecated rule IDs remain documented with their replacements for at least one major version.
