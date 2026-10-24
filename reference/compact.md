# Compact Mode

Status: Compact normative summary
Source: [core/00-rules.md](core/00-rules.md)

Use this file when the model context is small or when the task is a focused naming review. It is a summary, not a replacement for the core rules. If a conflict exists, [core/00-rules.md](core/00-rules.md) prevails.

## 1. Rules

| Rule | Requirement |
| --- | --- |
| R1 | CRUD uses exactly `insert`, `find`, `update`, `remove`. |
| R2 | Explicit conditions use `By<Condition>` after scope, in parameter order, without `Or`. A single `updateOne` may rely on a required aggregate id. |
| R3 | Members do not repeat the aggregate noun in the enclosing type. |
| R4 | `find` tolerates absence; `tryGet` returns an explicit result; `get` treats absence as an error. Priority: `find` > `tryGet` > `get`. |
| R5 | `One` = at most one; `Many` = zero or more. `Many` is preferred over `All`. `findMany()` returns the full declared scope. Bulk mutations require a condition and a guard. |
| R6 | Order: verb, scope, condition, transport suffix. Casing follows the language. |
| R7 | Collections are never null; `get` never returns absence; errors are typed and consistent. |
| R8 | Adaptations and waivers are recorded; core-rule waivers produce the partial profile. |
| R9 | Emission writes one idempotent managed block and preserves everything outside it. |
| R10 | Audit -> plan -> explicit approval -> execute -> validate. No source change before approval. |

## 2. Naming

```text
insertOne, insertMany
findOneById, findOneByEmail
findMany, findManyByStatus
updateOneById (preferred), updateOne(user) (identity-bearing)
updateManyByStatus
removeOneById, removeManyByStatus
```

Invalid:

```text
createUser, save, getUserById, listUsers, deleteUserById
findById (default profile), findByStatus, findManyUsers
findOneUserById
updateOne(patch), updateOne(), updateMany(), removeMany()
findManyByStatusOrTenantId
```

`updateOne(user)` and `updateOne(id, patch)` are valid only when the required parameter carries the aggregate root's unique id. `updateOne(patch)` and `updateOne()` are invalid. `removeOne`, `updateMany`, and `removeMany` always require a condition.

## 3. Read semantics

| Family | Absence | Result |
| --- | --- | --- |
| `findOneBy...` | Normal | Record or absence value |
| `findMany...` | Normal | Non-null collection; empty when none |
| `tryGetOneBy...` | Explicit | Success-or-absence result, no exception |
| `getOneBy...` | Error | Record or typed not-found error |

The repository default is `find`. `get` and `tryGet` belong in the service or boundary layer unless an R8 opt-in exists. Do not return `null` for a collection. Do not return `null` from `get`. Do not convert a failure into absence.

## 4. All-based adaptation

A project may replace `One`/`Many` for read, update, and removal scope under R8:

| Default | All-based |
| --- | --- |
| `findOneById` | `findById` |
| `findManyByStatus` | `findAllByStatus` |
| `findMany()` | `findAll()` |
| `updateOneById` | `updateById` or `update(user)` identity-bearing |
| `removeOneById` | `removeById` |

`insertOne` and `insertMany` are retained. `findByStatus` remains prohibited. This is an adaptation, not a core waiver, if cardinality remains distinguishable and `findAll()` returns the full scope.

## 5. Audit checklist

- Bound scope and exclusions.
- Inspect contracts, implementations, call sites, tests, errors, and constraints.
- Cite a file and symbol for every finding.
- Check R1–R10.
- Inspect `updateOne(user)` rather than flagging it mechanically.
- Classify severity: Blocker, High, Medium, Low.
- Report a candidate level, not a supported label.
- Do not edit source.

## 6. Plan checklist

- Feasibility: full, partial, or not feasible, with reasons.
- Benefits and drawbacks tied to evidence.
- Breaking changes: source, binary, public API, public behavior, serialized data, reflection/DI, configuration, documentation, tests.
- Ordered phases with owner, effort, compatibility, verification, rollback.
- Target architecture: layer boundaries, naming, visibility scope, ordering, absence/error model, enforcement.
- Risks with likelihood, impact, score, trigger, contingency.
- End with an explicit approval request.

## 7. Confirmation gate

```text
STOP. No source modification occurs before the approver records:
- the approval statement;
- the approved scope;
- the approved breaking-change classification;
- conditions;
- approver and date.
```

Silence, "looks fine", or a request for more detail is not approval. A material scope change requires a new approval.

## 8. Emission

Target order: explicit path -> `AGENTS.md` -> `CLAUDE.md` -> ask.

Markers:

```markdown
<!-- BEGIN crud-scope-standardizer v1 -->
...
<!-- END crud-scope-standardizer -->
```

Emit one block. Replace the existing region in place; preserve outside content and line endings; stop on multiple regions; re-read and verify. The block states profile, scope, rules, gate, register, and enforcement. A strict block says no decisions exist; an adapted block references its decision IDs.

## 9. Validation

- Naming scan and compiler pass.
- Contract tests cover absence, empty collections, updates, removals, and guards.
- `findMany` never returns null; `get` never returns absence.
- `updateOne` identity is required when `By` is absent.
- Bulk mutations reject empty conditions.
- API diff contains only approved changes.
- Managed block is singular and idempotent.
- Evidence supports the label.

## 10. Stop conditions

- Approval is absent or ambiguous.
- A core rule would be waived without a partial label.
- An All-based mapping is incomplete or ambiguous.
- A bulk mutation cannot be bounded.
- The target block is duplicated or unreadable.
- Evidence is insufficient for a verdict.
- The request requires exposing secrets.
