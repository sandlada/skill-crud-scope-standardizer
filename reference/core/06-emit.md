# Rule Emission and Adaptation

Status: Normative procedure
Source: [00-rules.md](00-rules.md)
Related: [04-plan.md](04-plan.md), [07-validate.md](07-validate.md)

Emission writes the adopted contract into a project instruction file. It owns one managed block and preserves everything outside it. Emission is a write and requires an explicit instruction and a confirmed target.

## 1. Target selection

Select in order:

1. an explicit path from the user;
2. `AGENTS.md` at the project root;
3. `CLAUDE.md` at the project root;
4. ask and do not guess.

If both `AGENTS.md` and `CLAUDE.md` exist and no target is chosen, ask. If both are chosen, both blocks must carry the same profile, decision IDs, and rule text. Do not update one silently when they diverge.

A custom target must be a Markdown instruction file or a documented equivalent. Do not emit into source, generated files, lockfiles, or an unreadable format. If the target is read-only or a symlink outside the project, stop and report.

## 2. Decision interview

| Decision | Default | Question |
| --- | --- | --- |
| Profile | Strict | Default vocabulary or an adaptation? |
| Cardinality | `One`/`Many` | Accept default, `Single`/`Multiple`, or All-based? |
| `All` scope | Prohibited | If `Many` is replaced, what is the complete mapping? |
| Single update | `updateOneById` preferred; bare identity-bearing allowed | Must every update name the condition, or may the aggregate id carry it? |
| Repository `get` | Not exposed | Is an R8 opt-in required? |
| Mutation returns | Boolean for one, count for many | Entity return or counts? |
| Async suffix | Language convention | Required, optional, or forbidden? |
| Acronym casing | Language convention | `Id`, `ID`, or another convention? |
| Enforcement | Naming scan plus review | Which CI check or analyzer? |
| Register location | `docs/crud-scope-decisions.md` | Where is the register stored? |

Record the answers in the register and reflect them in the block. A configurable choice produces an adapted profile. A core-rule waiver produces a partial profile and cannot be labeled compliant.

## 3. Profiles

### Strict

```text
insertOne, insertMany
findOneById, findMany, findManyByStatus
updateOneById (preferred), updateOne (identity-bearing)
updateManyByStatus
removeOneById, removeManyByStatus
```

`All` is prohibited. `get` and `tryGet` are service or boundary families in this profile. A candidate strict label is available after audit; the supported label requires validation.

### All-based adaptation

The project replaces `One`/`Many` for read, update, and removal scope. `insertOne` and `insertMany` are retained.

| Operation | Default | All-based |
| --- | --- | --- |
| Single read by id | `findOneById` | `findById` |
| Single read by unique condition | `findOneByEmail` | `findByEmail` |
| All records | `findMany()` | `findAll()` |
| Filtered plural read | `findManyByStatus` | `findAllByStatus` |
| Single update | `updateOneById` | `updateById` (preferred) or `update(user)` (identity-bearing) |
| Bulk update | `updateManyByStatus` | `updateAllByStatus` |
| Single removal | `removeOneById` | `removeById` |
| Bulk removal | `removeManyByStatus` | `removeAllByStatus` |

`findByStatus` is prohibited. `findBy<UniqueCondition>` is reserved for uniqueness-guaranteeing single lookups. `findAll()` returns the full declared scope. `update(patch)`, `update()`, `remove()`, `updateAll()` without a condition, and `removeAll()` without a condition are invalid. This is an adaptation under R8, not a core waiver, provided R5's invariant remains.

### Single/Multiple adaptation

Replace `One`/`Many` with `Single`/`Multiple`; cardinality remains distinguishable. `insertOne`/`insertMany` are retained. A bare single update is permitted only with an identity-bearing parameter.

### Partial

A core rule is waived. The block must state `Compliance: Partial; not compliant`, list the waived rule IDs, and state the lost guarantee. It must not be labeled strict or adapted.

## 4. Managed block

The block is delimited by exactly:

```markdown
<!-- BEGIN crud-scope-standardizer v1 -->
...
<!-- END crud-scope-standardizer -->
```

It must cover:

1. declared profile, authority, and catalog version;
2. scope and exclusions;
3. adopted R1–R7 rules in the project vocabulary;
4. workflow gate from R10;
5. decision register location and IDs;
6. enforcement mechanism.

Keep the block rules-only. Move rationale and examples to the reference corpus. A target of 100–150 lines is recommended; the full library template may be longer, but a project block should stay focused.

### Strict block example

```markdown
<!-- BEGIN crud-scope-standardizer v1 -->
## Repository contract rules
Profile: Strict. Compliance: candidate Strict; supported label requires validation.
Authority: CRUD Scope Standardizer, rule catalog 1.0, `reference/core/00-rules.md`.

Scope: repository contracts in `src/**`; ORM entities and routes excluded.
Verbs: insert, find, update, remove only.
Inserts: insertOne, insertMany.
Conditions: By<Condition> after scope; no Or; parameters in name order.
Aggregate noun: do not repeat the aggregate named by the interface.
Reads: findOneBy... returns an absence value; findMany/findManyBy... return a non-null
collection; get/tryGet are service or boundary families in this profile.
Cardinality: One/Many; findMany() returns the declared full scope; All is prohibited.
Single update: updateOneById preferred. updateOne(user) or updateOne(id, patch) is
permitted only when the parameter carries the aggregate root's unique id; the repository
rejects a missing or invalid identity. updateOne(patch) and updateOne() are invalid.
Bulk safety: updateManyBy... and removeManyBy... require a condition and a runtime guard.
Decision register: docs/crud-scope-decisions.md; no decisions exist.
Workflow: audit, plan, explicit approval, execute, validate. No source change before approval.
Enforcement: naming scan in CI and review checklist.
<!-- END crud-scope-standardizer -->
```

## 5. Merge procedure

1. Read the target and preserve its bytes.
2. Locate markers outside code fences and inline code.
3. If exactly one region exists, replace only its content.
4. If none exists, append the block after a blank line.
5. If multiple regions exist, stop and ask before consolidating.
6. Stop on an unclosed marker, wrong order, or mixed malformed fences.
7. Preserve CRLF or LF and all content outside the block.
8. Write once, then re-read and verify one region, profile, register IDs, and no secrets.

Do not reformat unrelated content. A formatter run after emission is a separate approved action.

## 6. Idempotency and verification

Running the same emission twice against the same target and profile must produce a byte-identical file after the first run.

| Check | Pass condition |
| --- | --- |
| Region count | Exactly one |
| Marker pairing | One complete pair, same version |
| Profile alignment | Block matches register |
| Decision IDs | Referenced IDs exist; strict block states none |
| Rule completeness | R1–R7, R10 identity rule, enforcement |
| Outside content | No change |
| Line endings | Consistent |
| Secrets | None |
| Duplicate rules | None |
| Read-back | Content matches intent |

Use SHA-256 before and after the second emission to prove idempotency.

## 7. Conflicts and rollback

If a user-authored rule conflicts with the block, report the text, location, rule ID, and options: remove the conflicting rule, scope the block to exclude the area, or record a partial waiver. Do not overwrite silently.

If the requested profile contradicts a core rule, refuse or emit a partial block that states it is not compliant. Never label it strict or adapted.

Before writing, capture the original bytes. If verification fails, restore the original bytes exactly and confirm the hash. Do not format or bulk-edit a failed emission before restoration.

## 8. Emission checklist

- [ ] Target confirmed and canonical.
- [ ] Profile and candidate/supported status recorded.
- [ ] Register exists when an adaptation exists; otherwise the block says no decisions exist.
- [ ] All configurable choices reflected.
- [ ] Core rules present and not weakened.
- [ ] Identity rule and bulk guards accurate.
- [ ] Exactly one managed region.
- [ ] Outside content unchanged; line endings preserved.
- [ ] No secrets or duplicate rules.
- [ ] Idempotency and read-back pass.
- [ ] Emission report recorded.
