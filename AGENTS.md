# Repository Agent Contract: CRUD and Scope Naming

## About this file

This file is the portable agent contract for a project that adopts the CRUD Scope Standardizer. It is both a working instruction file and the emission template used by the skill.

The managed block is the operative contract and the only region the skill may rewrite. The surrounding notes record provenance, adoption, and maintenance and are not repository policy.

To adopt the contract, copy this file to the target project root as `AGENTS.md`, or use it as the source text for an emission into `AGENTS.md`, `CLAUDE.md`, or a project-specific instruction file. Before the first audit, review the declared profile and confirm the decision register location.

Source of truth: `reference/core/00-rules.md` in the CRUD Scope Standardizer library. The catalog prevails over this file. Version: rule catalog 1.0; managed block marker v1. An agent treats the managed block as normative and does not infer rules from the surrounding notes.

<!-- BEGIN crud-scope-standardizer v1 -->

## 1. Authority and activation

This block is normative for repository contracts and aggregate-scoped typed data-access interfaces in this project. It is activated when an agent creates, renames, reviews, migrates, or emits rules for such a contract, and whenever a compliance claim is made.

Precedence, highest first:

1. the core rules in this block and in the rule catalog;
2. the project decision register for configurable choices;
3. the default profile;
4. language and framework conventions for casing, suffixes, and transport types;
5. local formatting preferences.

A lower item MUST NOT override a higher item. If a conflict cannot be resolved by this order, stop and report it.

In scope: repository interfaces, traits, protocols, abstract classes, and the public surface of their implementations; service, gateway, and port interfaces whose type name identifies the aggregate; generated clients only when the generator is configured or wrapped by the project.

Out of scope: ORM entities, database schema objects, migration files, framework lifecycle methods, private helpers outside a contract, and transport route names, JSON keys, and payload fields, unless the project explicitly adopts the same vocabulary.

Declared profile: Strict, default vocabulary, rule catalog 1.0. Catalog pointer: `reference/core/00-rules.md` in the CRUD Scope Standardizer library; when this block is copied, record the pinned catalog location in the decision register. An audit reports a candidate level; the supported strict label requires validation evidence. Decision IDs affecting this block: none. Any adaptation requires a register entry and a corresponding edit inside this block.

## 2. Rules

### R1. CRUD verbs

CRUD operations MUST use exactly `insert`, `find`, `update`, and `remove`. Canonical forms: `insertOne`, `insertMany`, `findOneBy...`, `findMany`, `findManyBy...`, `updateOneBy...` (preferred), `updateOne` (identity-bearing), `updateManyBy...`, `removeOneBy...`, `removeManyBy...`.

Incorrect: `createUser`, `save`, `getUserById`, `listUsers`, `deleteUserById`.

Distinct lifecycle operations may keep their own verbs when documented and not CRUD synonyms. A composite verb such as `upsertOneById` requires a register entry. Framework-mandated methods are framework surface and are not renamed.

### R2. Conditions

Every explicit selecting condition MUST use `By<Condition>` after the scope and before any transport suffix, with domain predicates and parameters in name order. Multiple predicates join with `And`; `Or` MUST NOT appear in a name. Use `In`, `Between`, `Not`, `IsNull`, and `IsNotNull` for their standard meanings.

The aggregate's intrinsic identity is not an explicit condition. A single-aggregate `updateOne` MAY omit `By` when a required parameter carries the aggregate root's unique identifier. `updateOneById` is preferred. `updateOne(user)` and `updateOne(id, patch)` are valid; `updateOne(patch)` and `updateOne()` are invalid.

`removeOne`, `updateMany`, and `removeMany` MUST NOT omit a condition. A singleton aggregate may expose `findOne()` only when the contract guarantees exactly one instance and documents it. Complex boolean logic uses a specification or criteria object with one segment.

### R3. Aggregate noun

A member name MUST NOT repeat the aggregate noun established by the enclosing type.

Correct: `IUserRepository.findOneById(id)`, `IOrderRepository.findManyByCustomerId(customerId)`.

Incorrect: `IUserRepository.findOneUserById(id)`, `IUserRepository.findManyUsersByStatus(status)`.

A condition may contain another aggregate's noun when it refers to that aggregate. A cross-aggregate read should use a dedicated repository or read model.

### R4. Read semantics

Priority is `find`, then `tryGet`, then `get`.

- `findOneBy...` returns the record or an absence value; absence is not an exception.
- `findMany` and `findManyBy...` return a non-null collection, empty when there are no matches.
- `tryGetOneBy...` returns an explicit success-or-absence result and does not throw for absence.
- `getOneBy...` returns the record or a typed error; it never returns an absence value.
- `getManyBy...` is used only where the contract requires a non-empty result; otherwise use `findManyBy...`.

The default repository read surface exposes `find` only. `get` and `tryGet` belong at the service or boundary layer unless the project records a repository opt-in under R8. A `findOne` or `getOne` condition MUST guarantee uniqueness; otherwise use `findManyBy...` with a limit and documented order.

### R5. Cardinality and safety

Cardinality MUST be distinguishable. Default: `One` for at most one target; `Many` for zero or more. `Many` is preferred over `All`; `All` is prohibited in the default profile.

`findMany()` with no semantic parameters returns all records in the repository's declared visibility scope, with documented default ordering. `findManyBy<Condition>` returns all matches in that scope. Presentation and infrastructure parameters do not change the meaning of a parameterless operation.

Bulk update and removal MUST reject an empty or missing condition at runtime. An empty identifier collection in `removeManyByIdIn(ids)` is a no-op and is never interpreted as removing all records. The single-update identity exception does not relax bulk safety.

In a multi-tenant or policy-scoped repository, "all" means all records visible to the declared scope. The scope MUST be documented; hidden filters are prohibited.

A project MAY replace the `One`/`Many` vocabulary under R8, including an All-based profile, if cardinality remains distinguishable and a parameterless plural read returns the full scope. The replacement applies to read, update, and removal scope; `insertOne` and `insertMany` are retained. This is an adaptation, not a core waiver.

### R6. Grammar

Canonical order: `Verb Scope? Condition? TransportSuffix?`, with `Verb` from `Insert`, `Find`, `Update`, `Remove`; `Scope` from `One`, `Many` in the default profile, or the approved R8 cardinality vocabulary (for example, implicit single plus `All`); `Condition` as `By` terms joined by `And`; and a framework-mandated suffix such as `Async`. `insertOne` and `insertMany` retain their forms in every profile.

Casing follows the host language: `FindOneByIdAsync` in C#, `findOneById` in TypeScript, `find_one_by_id` in Python, `FindOneByID` in Go. A name MUST NOT contain the repository type name or the word `Repository`, place a transport marker inside the semantic segments, or abbreviate domain terms unless recorded. Ordering suffixes require an R8 entry.

A bare `updateOne` is permitted only with a required aggregate identity. A bare `findOne` is permitted only for a singleton aggregate. A bare `removeOne` is prohibited.

### R7. Return and error contracts

- `findOneBy...` returns the record or an absence value.
- `findMany` and `findManyBy...` return a non-null collection.
- `getOneBy...` returns the record or a typed error; it never returns an absence value.
- `tryGetOneBy...` returns an explicit result and does not throw for absence.
- `insertOne` returns the inserted record, its identifier, or both under a documented policy.
- `insertMany` returns the inserted records, identifiers, or affected count under the same policy.
- `updateOne`, `updateOneBy...`, and `removeOneBy...` return an affected count or a boolean.
- `updateOne` and `updateOneBy...` MAY return the updated record; `removeOneBy...` MUST NOT return a record.
- `updateManyBy...` and `removeManyBy...` return an affected count or equivalent bulk result.

Do not swallow infrastructure errors; translate them while preserving the distinction between absence and failure.

### R8. Adaptation and the decision register

A deviation from the default profile MUST be recorded as a decision. Core rules that MUST NOT be waived while claiming compliance are R1, R2, R3, R4, R5, R6, R7, and R10. R6 grammar is mandatory; the scope-token vocabulary and surface conventions such as casing and suffixes may vary when recorded.

Configurable choices include cardinality tokens, the `All` scope, repository `get`/`tryGet` placement, transport suffixes, acronym casing, mutation return policy, scalar helpers, ordering in names, compatibility strategy, emission target, and enforcement scope.

Each entry records: ID, Rule, Default, Decision, Rationale, Scope, Consequence, Approver, Date, Review. A core-rule waiver places the project in the partial profile. Replacing the cardinality vocabulary is an adaptation, not a waiver, provided R5's invariant remains. An unrecorded deviation is a defect. A register entry that conflicts with a core rule is invalid.

A strict project with no adaptation or waiver MAY omit the register, provided the emitted block states that no decisions exist.

### R9. Emission

Emission requires an explicit instruction and a confirmed target. Target order: an explicit path; `AGENTS.md`; `CLAUDE.md`; otherwise ask. If both files exist and no target is chosen, ask. If both are chosen, both blocks must match.

The emitted content is one managed region. Merge procedure:

1. Read the target.
2. If exactly one region exists, replace its content and preserve everything outside it.
3. If none exists, append the block after a blank line.
4. If multiple regions exist, stop and ask before consolidating.
5. Preserve line endings.
6. Do not rewrite or reformat unrelated content.
7. Re-read and verify one region, the profile, and register alignment.

Emission is idempotent and non-destructive. It does not delete user-authored rules; it reports conflicts. The block states the profile, applicable rules, confirmation gate, catalog pointer, register location, and enforcement mechanism.

### R10. Migration confirmation gate

| Phase | Mode | Allowed actions | Exit condition |
| --- | --- | --- | --- |
| A. Audit | Read-only | Inspect architecture, repositories, call sites, tests, constraints | Audit report |
| B. Plan | Read-only | Feasibility, benefits, drawbacks, breaking changes, steps, target architecture, risks, rollback | Plan presented |
| C. Confirmation | No writes | Ask for explicit approval, answer questions, revise | Approval recorded |
| D. Execution | Writes allowed | Apply the approved plan in reviewed increments | Each increment verified |
| E. Validation | Approved fixes and reports | Run checks, produce a validation report | Acceptance criteria met |

No source modification may occur before explicit confirmation. Silence, a request for more detail, or an ambiguous comment is not approval. A material change to scope, target architecture, or breaking-change classification requires a new confirmation.

## 3. Mandatory workflow

1. Bound the scope and read the architecture.
2. Produce an evidence-based audit with a candidate level.
3. Produce the six-part plan with risks and rollback.
4. STOP and request explicit confirmation.
5. Execute only the approved scope in reviewed increments.
6. Validate against the plan and issue the supported label.
7. Report artifacts, unresolved conflicts, and the next permitted action.

## 4. Decision register

Register location: `docs/crud-scope-decisions.md`. The register is the only mechanism for adapting this contract. If decisions are stored elsewhere, record the actual location in this block and at the top of that file before claiming an adaptation.

The register contains one entry per decision with the fields in R8. Core rules prevail over the register. A core-rule waiver produces a partial profile and prevents a compliance claim. A decision that changes the emitted rules must be reflected in this block in the same change.

## 5. Emission and enforcement

Emission is idempotent, scoped, and non-destructive. It replaces one managed region, preserves all content outside it and line endings, and stops when multiple regions exist. It requires explicit instruction and a confirmed target; during a migration it occurs only after R10 approval.

Enforcement is part of adoption: configure a naming scan in CI and a review checklist for repository changes. A rule that is not checked is documentation, not enforcement.

## 6. Compliance labels

| Level | Definition | Permitted label |
| --- | --- | --- |
| Strict | All core rules satisfied; default profile; no adaptations or waivers | Compliant with CRUD Scope Standardizer, strict profile |
| Adapted | All core rules satisfied; configurable choices recorded | Compliant with CRUD Scope Standardizer, adapted profile |
| Partial | One or more core rules waived or violated, with a recorded waiver | Partially aligned; not compliant |
| Non-compliant | Core rules violated without a recorded waiver | Not compliant |
| Audit-only | No migration or emission performed | Audited only; compliance not asserted |

An audit reports a candidate level. The supported label requires validation evidence. A project MUST NOT be described as compliant if any core rule is waived.

<!-- END crud-scope-standardizer -->

## Maintaining the managed block

The delimited region is the idempotent emission region. The skill owns only that region; everything outside it is provenance and maintenance text and must survive an emission unchanged.

- Replace the region in place. Do not append a second copy when one exists.
- Preserve line endings and all content outside the region.
- Stop and ask when the target contains more than one managed region.
- Keep the marker version aligned with the catalog's major version.
- After writing, re-read and verify exactly one region and a profile matching the decision register.
- Report conflicts with user-authored instructions instead of overriding them.
- Do not place secrets, credentials, tokens, or project-sensitive data inside the region.

## Adoption notes

- The declared profile is the source of the emitted compliance status. Changing it without a register entry is a defect.
- The decision register path is a default. A project that stores decisions elsewhere must record the actual location before claiming an adaptation.
- The reference pointer inside the block names the catalog by its library path. When the block is copied into another project, record the absolute or pinned location in the register.
- If this file is used as `CLAUDE.md`, the same block applies; do not maintain divergent profiles across instruction files.
