---
name: crud-scope-standardizer
description: Audits, plans, emits, migrates, and reviews repository CRUD and scope naming. Use for repository interfaces, traits, protocols, or aggregate-scoped data-access contracts that need insert/find/update/remove verbs, By<Condition> filters, One/Many or adapted cardinality, distinct find/tryGet/get semantics, a governed migration, or managed-block emission into a project instruction file.
---

# CRUD Scope Standardizer

## Purpose

This skill standardizes repository and aggregate-scoped data-access contracts. It covers CRUD verbs, condition encoding, cardinality, method-name grammar, and absence semantics. It produces evidence-based audits, approval-gated migration plans, governed migrations, idempotent rule emission, and validation reviews.

The normative source is [reference/core/00-rules.md](reference/core/00-rules.md). Every other file is subordinate. When a conflict exists, the core rules prevail.

The rules apply to repository interfaces, traits, protocols, abstract classes, and the public surface of their implementations; to service, gateway, and port interfaces whose type identifies the aggregate; and to generated clients only when the project configures or wraps the generator.

The rules do not apply to ORM entities, schema objects, migration files, framework lifecycle methods, private helpers outside a contract, or transport routes, JSON keys, and payload fields unless the project explicitly adopts the same vocabulary.

## When to use

- Auditing repository contracts for verb drift, missing `By` markers, ambiguous cardinality, or conflated read semantics.
- Planning a rename or restructuring before code changes.
- Migrating an approved plan in reviewed increments.
- Emitting the rules into `AGENTS.md`, `CLAUDE.md`, or a custom instruction file.
- Reviewing an interface, change set, or compliance claim.
- Establishing or amending a decision register.

## When not to use

- Routine feature work with no repository-contract impact.
- Formatting or lint-only changes.
- Renaming ORM entities, schema objects, routes, or serialized keys.
- Framework lifecycle methods that must not be renamed.
- A request to edit contracts immediately without an audit, plan, and explicit confirmation.
- A codebase with no aggregate-scoped data-access contract.

## Core contract

| Rule | Requirement |
| --- | --- |
| R1 | CRUD uses exactly `insert`, `find`, `update`, `remove`. |
| R2 | Explicit conditions use `By<Condition>` after scope, in parameter order, without `Or`. A single aggregate `updateOne` may rely on a required aggregate id; `updateOneById` is preferred. |
| R3 | Members do not repeat the aggregate noun established by the enclosing type. |
| R4 | `find` tolerates absence; `tryGet` returns an explicit result; `get` treats absence as an error. Priority: `find` > `tryGet` > `get`. |
| R5 | `One` = at most one; `Many` = zero or more. `Many` is preferred over `All`; `findMany()` returns the declared full scope. Bulk mutations require a condition and a runtime guard. |
| R6 | Order: verb, scope, condition, transport suffix. Casing follows the host language. |
| R7 | Collections are never null; `get` never returns absence; errors are typed and consistent. |

R8 (adaptation), R9 (emission), and R10 (migration gate) are core process rules. A configurable adaptation produces the adapted profile. A core-rule waiver produces the partial profile and cannot be labeled compliant.

## Context budget and loading protocol

Load only what the task needs. Do not read the entire `reference/` directory. Default cap: the skill entrypoint plus up to five core files, or roughly 20k tokens of skill material. Load one template or one example only when the task needs it.

| Mode | Must read | Optional | Budget |
| --- | --- | --- | ---: |
| Quick review | this file, or [compact.md](reference/compact.md) for a small context | none | 3–6k |
| Audit | [core/00-rules.md](reference/core/00-rules.md) R1–R10 + [core/03-audit.md](reference/core/03-audit.md) | `core/01` or `core/02` for a grammar or absence dispute | 8–10k |
| Plan | `core/00` R10 + [core/04-plan.md](reference/core/04-plan.md) | `core/05` for rollback detail | 8–10k |
| Emit | `core/00` R8–R9 + [core/06-emit.md](reference/core/06-emit.md) | `core/07` for validation | 7–9k |
| Migrate | `core/00` + `core/03` + `core/04` + `core/05` + `core/07` | one template and one example | 15–20k |
| Review | `core/00` + [core/07-validate.md](reference/core/07-validate.md) | `core/02` for read semantics | 6–8k |

Loading rules:

1. Start with this file. Do not preload references.
2. Read core files one at a time; do not load the whole guide set.
3. Prefer a heading or `rg` result over a full-file read.
4. Read `guides/00-rationale.md` only when the user asks why or an edge case needs resolution.
5. Read one language template from `guides/templates/`; never load all seven.
6. Read one example from `guides/examples/`; never load both by default.
7. Read `guides/10-glossary.md` only for terminology conflicts.
8. If the context is small, use `compact.md` and avoid guides unless blocked.

## Operating modes

The mode follows the user's intent. When unclear, default to Audit. Audit, Plan, and Review are read-only with respect to source.

| Mode | Write access | Purpose | Exit artifact |
| --- | --- | --- | --- |
| Audit | Read-only | Inventory contracts, call sites, tests, constraints; assess R1–R10 | Audit report |
| Plan | Read-only | Convert findings into a feasibility-assessed plan | Migration plan, unapproved |
| Emit | Managed block only | Install or update rules in a confirmed file | Verified block |
| Migrate | Approved scope only | Audit through validation | Migrated contracts and report |
| Review | Read-only | Assess a change, proposal, or claim | Review report |

Audit never edits or emits. Plan is not approval. Emit requires an explicit instruction and confirmed target. Migrate is the only mode that changes source, and only within approved scope.

## Mandatory workflow

1. **Bound the scope.** Read the core rules. Identify target modules, aggregate contracts, language conventions, and exclusions.
2. **Read the architecture.** Inspect contracts, implementations, services, call sites, tests, errors, and constraints. Record file and symbol evidence.
3. **Audit.** Report per-rule status, findings with severity, register state, and a candidate compliance level. Do not propose edits as applied changes.
4. **Plan.** Cover feasibility, benefits, drawbacks, breaking changes, steps, and target architecture, plus risks and rollback.
5. **Stop.** Present the plan and request explicit confirmation. No source change occurs before approval. Silence or ambiguity is not approval.
6. **Execute.** Apply the approved plan in reviewed increments, each with its stated verification. A material change requires a new confirmation.
7. **Validate.** Run the agreed checks, verify the target architecture, confirm the managed block when applicable, and close or reopen findings.
8. **Report.** State the mode, artifacts, supported label, and unresolved conflicts. Do not claim success without evidence.

## Hard rules

- No source modification before explicit approval of the migration plan (R10).
- Audit and Plan are read-only.
- Silence, a request for more detail, or an ambiguous comment is not approval.
- A waived or violated core rule prevents a compliance claim.
- Emission writes only inside one managed block and preserves everything outside it.
- Core rules prevail over the register; the register prevails over the default profile and local conventions.
- An unrecorded deviation is a defect: report it, do not implement it.
- Do not rename framework methods; document them as framework surface.
- Do not expose credentials, tokens, secrets, or raw database errors.
- Do not report validation success without evidence from the current review.
- Do not load the full reference corpus; follow the loading protocol.

## Output artifacts

| Artifact | Mode | Required content | Write policy |
| --- | --- | --- | --- |
| Audit report | Audit, Migrate | Scope, inventory, findings, severity, candidate level, evidence | Returned; written on request |
| Migration plan | Plan, Migrate | Six plan sections, risks, rollback, approval request | Returned; written on request |
| Decision entry | Plan, Migrate | ID, rule, default, decision, rationale, scope, consequence, approver, date, review | Proposed; written after approval |
| Managed block | Emit | Profile, rules, gate, register, enforcement | Only inside the block after target confirmation |
| Validation report | Migrate, Review | Checks, results, findings, residual risk, label | Returned; written on request |
| Review report | Review | Verdict, findings, required changes, evidence | Returned; written on request |

## Reference map

| File | Use for |
| --- | --- |
| [reference/compact.md](reference/compact.md) | Small-context or quick-review summary |
| [reference/core/00-rules.md](reference/core/00-rules.md) | Normative R1–R10 |
| [reference/core/01-grammar.md](reference/core/01-grammar.md) | Grammar, conditions, edge cases |
| [reference/core/02-read-semantics.md](reference/core/02-read-semantics.md) | `find` / `tryGet` / `get` |
| [reference/core/03-audit.md](reference/core/03-audit.md) | Audit procedure and report |
| [reference/core/04-plan.md](reference/core/04-plan.md) | Six-part plan and gate |
| [reference/core/05-execute.md](reference/core/05-execute.md) | Execution, compatibility, rollback |
| [reference/core/06-emit.md](reference/core/06-emit.md) | Emission and adaptation |
| [reference/core/07-validate.md](reference/core/07-validate.md) | Validation and labels |
| [reference/guides/00-rationale.md](reference/guides/00-rationale.md) | Rationale and edge cases |
| [reference/guides/templates/index.md](reference/guides/templates/index.md) | Choose one language template |
| [reference/guides/examples/index.md](reference/guides/examples/index.md) | Choose one worked example |
| [reference/guides/10-glossary.md](reference/guides/10-glossary.md) | Terminology |

## Quick naming reference

| Operation | Canonical default | Rejected |
| --- | --- | --- |
| Create one | `insertOne` | `createUser`, `add`, `save` |
| Create many | `insertMany` | `createMany`, `saveAll` |
| Read one by condition | `findOneBy<Condition>` | `getUserById`, `findUserById`, `findById` in the default profile |
| Read all in scope | `findMany()` | `findAll()`, `list()`, `getAll()` |
| Read many by condition | `findManyBy<Condition>` | `findByStatus`, `findMany(status)` |
| Update one | `updateOneBy<Condition>`; `updateOne` with an aggregate id | `modify`, `patch`, `updateOne(patch)`, `updateOne()` |
| Update many | `updateManyBy<Condition>` | `updateAll`, `bulkUpdate` |
| Remove one | `removeOneBy<Condition>` | `deleteById`, `destroy`, `drop` |
| Remove many | `removeManyBy<Condition>` | `deleteAll`, `purgeAll` |
| Complex filter | `findManyBySpecification` | `findManyByStatusOrTenantId` |

An approved All-based adaptation maps `findOneById` to `findById`, `findMany()` to `findAll()`, and plural filters to `findAllBy...`; `insertOne` and `insertMany` are retained. See [core/06-emit.md](reference/core/06-emit.md). `get` and `tryGet` are service or boundary families by default; a repository may expose them only as a recorded R8 opt-in. A bare single update is permitted only with a required aggregate id; `updateOneById` is preferred.

## Stop conditions

Stop and report, without writing, when:

- a register entry conflicts with a core rule;
- the emission target has multiple managed blocks;
- approval is absent, ambiguous, conditional, or narrower than the proposed change;
- feasibility is not feasible;
- the target architecture cannot be read;
- the user asks to skip the audit or gate;
- the action would modify an out-of-scope surface;
- evidence is insufficient for a verdict;
- the emission target cannot be determined;
- the request requires exposing secrets.

## Quality bar

- Every finding cites a file and symbol.
- Every verdict is pass, fail, not applicable, or waived.
- Examples use the target language and adopted vocabulary.
- Plans separate evidence from recommendation and label estimates.
- A label is assigned only after all core rules are checked.
- Artifacts contain no secrets or raw database errors.
- Excluded surfaces are named, not silently skipped.
- The same inputs and register produce the same verdicts on repeated runs.
