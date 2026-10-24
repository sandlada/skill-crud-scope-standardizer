# CRUD Scope Standardizer

A production-oriented Agent Skill for standardizing repository CRUD and scope naming across languages. It audits existing contracts, plans and executes governed migrations, and emits project-specific rules into `AGENTS.md`, `CLAUDE.md`, or a custom instruction file.

The skill is language-agnostic. The normative rules are in [reference/core/00-rules.md](reference/core/00-rules.md). The portable contract is [AGENTS.md](AGENTS.md). A compact summary is available at [reference/compact.md](reference/compact.md).

## The problem

Repository layers drift. One aggregate accumulates `createUser`, `getUserById`, `getAllUsers`, `deleteUser`, `findUsersByStatus`, and aliases added by the next feature. Names hide whether absence is normal or an error. Cardinality is ambiguous. Interface names repeat the entity on every member. Reviews cannot agree because no rule was written down.

This skill replaces that drift with a small contract:

- four CRUD verbs: `insert`, `find`, `update`, `remove`;
- explicit conditions using `By<Condition>`;
- no repeated aggregate noun inside a typed contract;
- distinct `find`, `tryGet`, and `get` semantics;
- explicit `One`/`Many` cardinality, with `findMany()` meaning all in the declared scope;
- a governed path from audit to approved migration.

## What it provides

| Capability | Outcome |
| --- | --- |
| Audit | Evidence-based inventory of contracts, call sites, violations, and constraints |
| Migration plan | Feasibility, benefits, drawbacks, breaking changes, steps, target architecture, risks, rollback |
| Confirmation gate | A hard stop before any source modification; approval is explicit and recorded |
| Migration execution | Slice-by-slice changes with compatibility, verification, and rollback |
| Rule emission | One idempotent managed block for `AGENTS.md`, `CLAUDE.md`, or a custom path |
| Adaptation | A decision register for project choices, including rejecting `One`/`Many` |
| Validation | Naming scans, semantic tests, acceptance criteria, and compliance labels |

## Repository layout

```text
.
├── SKILL.md                          # Entrypoint and workflow
├── AGENTS.md                         # Portable contract and emission template
├── README.md                         # This document
└── reference/
    ├── compact.md                    # Small-context summary
    ├── core/
    │   ├── 00-rules.md               # Normative R1-R10
    │   ├── 01-grammar.md             # Grammar and conditions
    │   ├── 02-read-semantics.md      # find / tryGet / get
    │   ├── 03-audit.md               # Audit procedure
    │   ├── 04-plan.md                # Six-part plan and gate
    │   ├── 05-execute.md             # Execution and rollback
    │   ├── 06-emit.md                # Emission and adaptation
    │   └── 07-validate.md            # Validation and labels
    └── guides/
        ├── 00-rationale.md           # Why the rules exist; edge cases
        ├── 10-glossary.md            # Terminology
        ├── templates/
        │   ├── index.md
        │   ├── typescript.md
        │   ├── csharp.md
        │   ├── java.md
        │   ├── kotlin.md
        │   ├── python.md
        │   ├── go.md
        │   └── rust.md
        └── examples/
            ├── index.md
            ├── typescript.md
            └── csharp.md
```

## Installation

Copy the entire directory into the skill location and keep the relative structure. `SKILL.md` is the entrypoint; `reference/` is loaded on demand.

### OpenCode

Project-local:

```text
<project>/.opencode/skills/crud-scope-standardizer/
```

Global:

```text
~/.config/opencode/skills/crud-scope-standardizer/
```

### Claude Code

```text
<project>/.claude/skills/crud-scope-standardizer/
```

### Generic runtimes

Load `SKILL.md` as the entrypoint and make `reference/` readable. If the runtime supports always-on instruction files, use [AGENTS.md](AGENTS.md) or emit its managed block into the project's existing file.

## Context loading

The library is intentionally layered. Do not load the whole reference corpus.

| Task | Read | Approximate budget |
| --- | --- | ---: |
| Quick naming review | `SKILL.md` or `reference/compact.md` | 3–6k tokens |
| Audit | `core/00` + `core/03` | 8–10k |
| Plan | `core/00` R10 + `core/04` | 8–10k |
| Emit | `core/00` R8–R9 + `core/06` | 7–9k |
| Migrate | `core/00` + `core/03` + `core/04` + `core/05` + `core/07` + one template | 15–20k |
| Review | `core/00` + `core/07` | 6–8k |

Rules of thumb:

- read only the core files required by the mode;
- load one language template, never all seven;
- load one example, not the whole examples directory;
- use `rg` or a heading lookup instead of reading a whole guide;
- use `compact.md` when the context window is small.

## Quick start

### Audit

```text
Use the crud-scope-standardizer skill to audit the repository layer in this project.
Read-only. Cite a file and symbol for every finding, assess R1-R10, and report a
candidate compliance level. Do not modify source.
```

### Plan

```text
Using the audit, produce a migration plan. Cover feasibility, benefits, drawbacks,
breaking changes, ordered steps, target architecture, risks, and rollback. Stop after
presenting the plan; do not modify source until I approve it.
```

### Approve and execute

```text
Approve the migration plan as presented. Proceed with the approved phases. Record the
approval and stop if the scope or breaking-change classification changes.
```

### Emit

```text
Emit the CRUD Scope Standardizer rules into AGENTS.md using the strict profile. Replace
the existing managed block in place and preserve all content outside it.
```

### Review

```text
Review this repository interface against the CRUD Scope Standardizer. Check R1-R7,
absence semantics, cardinality, update identity, and bulk guards. Do not modify source.
```

## The rules in one page

### R1. Four CRUD verbs

Use `insert`, `find`, `update`, and `remove`. Do not use `create`, `save`, `get`, `list`, `modify`, `delete`, or their synonym families for repository CRUD.

```text
insertOne(user)
findOneById(id)
updateOneById(id, patch)
removeOneById(id)
```

### R2. Conditions use `By`

```text
findManyByStatus(status)
findManyByStatusAndTenantId(status, tenantId)
findManyByIdIn(ids)
removeOneByEmail(email)
```

A single-aggregate `updateOne` may omit `By` when a required parameter carries the aggregate root's unique id. `updateOneById` is preferred.

```text
updateOne(user)              // aggregate id is the implicit condition
updateOne(id, patch)         // id parameter is the implicit condition
updateOneById(id, patch)     // preferred
updateOne(patch)             // invalid: no identity
updateOne()                  // invalid: no target
```

### R3. Do not repeat the aggregate noun

```text
IUserRepository.findOneById(id)                    correct
IUserRepository.findOneUserById(id)                incorrect
IOrderRepository.findManyByCustomerId(customerId)  correct: another aggregate
```

### R4. `find` > `tryGet` > `get`

| Family | Absence | Typical result |
| --- | --- | --- |
| `findOneBy...` | Normal | Record or absence value |
| `findMany...` | Normal | Non-null collection, empty when none |
| `tryGetOneBy...` | Explicit result | Success-or-absence without throwing |
| `getOneBy...` | Error | Record or typed not-found error |

The repository default is `find`. `get` and `tryGet` belong in the service or boundary layer where absence has a business meaning.

### R5. Cardinality is explicit

`One` means at most one; `Many` means zero or more. `Many` is preferred over `All`. `findMany()` with no semantic parameters returns all records in the declared scope.

```text
findMany()
findManyByStatus(status)
updateOneById(id, patch)
updateManyByStatus(status, patch)
```

Bulk mutation without a condition is prohibited. `updateOne` is the only mutation that may omit `By`, and only with an identity-bearing parameter.

### R6. Fixed grammar

```text
Verb + Scope + By<Condition> + TransportSuffix
```

Casing follows the host language: `FindOneByIdAsync`, `findOneById`, `find_one_by_id`, `FindOneByID`.

### R7. Return contracts are observable

Collections are never null. `find` absence is a value. `get` absence is an error. Mutation returns are consistent and documented.

### R8-R10. Adaptation, emission, and the gate

- Every configurable choice is recorded in the decision register.
- Emission writes one idempotent managed block.
- Migration requires an audit, a plan, and explicit approval before any source change.

## Adaptation

A project may adopt the skill without accepting every default. Core rules remain; the configurable surface can change.

Example: the project rejects `One`/`Many` and uses `All`.

| Default | Adapted |
| --- | --- |
| `findOneById` | `findById` |
| `findManyByStatus` | `findAllByStatus` |
| `findMany()` | `findAll()` |
| `updateOneById` | `updateById` |
| `removeManyByStatus` | `removeAllByStatus` |

`insertOne` and `insertMany` are retained. The mapping is recorded in the decision register, the emitted block reflects it, and ambiguous forms such as `findByStatus` remain prohibited. This is an adapted profile, not a partial one, because no core rule is waived.

## Validation

Validation is not a naming scan alone. It covers naming, absence and error semantics, return contracts, call sites, bulk guards, adaptation alignment, tests, API diff, and enforcement.

See [reference/core/07-validate.md](reference/core/07-validate.md) for the checklist and report template.

## Supported languages

The grammar is language-neutral. Templates cover TypeScript, C#, Java, Kotlin, Python, Go, and Rust. Examples cover TypeScript and C#.

## Design principles

1. **Evidence over opinion.** Findings cite a file and symbol.
2. **Names carry contracts.** A name states the verb, cardinality, and condition.
3. **Absence is not an exception by default.** `find` keeps absence in the type system.
4. **One aggregate, one identity.** A single update may rely on the aggregate id; bulk operations may not.
5. **Adaptation is explicit.** A deviation is recorded, never inferred.
6. **No migration without consent.** The gate is a hard stop.
7. **Emission is idempotent.** One managed block, preserved surroundings.
8. **Load on demand.** The full corpus is a handbook, not a prompt.

## FAQ

**Is `get` banned?**  
No. `get` has a distinct contract: absence is an error. It is not a repository CRUD synonym and is not the default read family.

**Is `updateOne` allowed without `ById`?**  
Yes, when a required parameter carries the aggregate root's unique identifier. `updateOneById` is preferred. `updateOne(patch)` and `updateOne()` are invalid.

**Can we keep `findAll`?**  
Only under an approved All-based adaptation that replaces the `One`/`Many` vocabulary, with a complete mapping in the decision register. `findByStatus` remains ambiguous and prohibited.

**Does this rename database columns or JSON keys?**  
No. The rules apply to repository contracts and aggregate-scoped interfaces. ORM entities, schema objects, routes, and payload keys are outside the default scope.

**Can we migrate incrementally?**  
Yes. The plan defines phases, compatibility shims, per-phase verification, and rollback. A hard cut is allowed only when the plan justifies it.

**What if we disagree with a rule?**  
Record a decision. A configurable choice produces an adapted profile. A core-rule waiver produces a partial profile and cannot be labeled compliant.

**Does the skill modify code during an audit?**  
No. Audit and Plan are read-only. Execution begins only after explicit approval.

## Versioning

The rule catalog is version 1.0 and the managed block marker is `v1`. A core rule change requires a major version. A configurable choice or example change requires a minor version. Deprecated rule IDs remain documented with their replacements for at least one major version.

## Contributing

1. Read the core rules before changing any document.
2. Keep rule IDs, terminology, and examples consistent across all files.
3. Update the managed block version when a rule change affects emission.
4. Add or update a validation check for every new enforceable rule.
5. Do not add placeholders, TODOs, or claims without evidence.
6. Run the link, size, and consistency checks before reporting completion.
