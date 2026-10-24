# Validation and Review

Status: Normative procedure
Source: [00-rules.md](00-rules.md)
Related: [03-audit.md](03-audit.md), [05-execute.md](05-execute.md), [06-emit.md](06-emit.md)

Validation is evidence-based. A compiler-green build is necessary but not sufficient: naming, absence semantics, and runtime guards are not all visible to the compiler.

## 1. Dimensions

| Dimension | Question | Evidence |
| --- | --- | --- |
| Naming | Do names follow R1–R3, R5, R6? | Contract inspection, scan, AST |
| Semantics | Do `find`, `tryGet`, `get` behave per R4? | Absence, multi-match, failure tests |
| Contracts | Do returns and errors follow R7? | Signatures, error types, tests |
| Call sites | Do callers use the target and handle absence? | Compiler, search, route tests |
| Guards | Do bulk mutations reject empty conditions? | Guard tests |
| Adaptation | Is every deviation recorded and emitted? | Register, block comparison |
| Emission | Is there one block with the right profile? | Marker count, read-back, idempotency |
| Tests | Do characterization and target tests pass? | Test report |
| Enforcement | Will the next violation be caught? | CI, analyzer fixture, review checklist |

## 2. Scan scope and patterns

Scope the scan to repository contracts, aggregate-scoped interfaces, public API files, generated contract snapshots, and the managed block. A repository-wide `get` scan produces noise.

| Pattern | Candidate finding | False positive |
| --- | --- | --- |
| `\b(create\|add\|save\|store\|persist)[A-Z]\w*\s*\(` | Synonym create | Domain factory |
| `\b(get\|load\|read\|fetch\|list\|retrieve\|select)[A-Z]\w*\s*\(` | Synonym read in a repository | Service `get` is valid |
| `\b(modify\|patch\|set\|change\|edit\|apply)[A-Z]\w*\s*\(` | Synonym update | Patch helper outside a repository |
| `\b(delete\|destroy\|drop\|erase\|purge\|discard)[A-Z]\w*\s*\(` | Synonym remove | Framework or migration |
| `\bfind(One\|Many)?\s*\(` | Bare `find` | Internal query helper |
| `\bfindBy[A-Z]\w*\s*\(` | Unscoped `findBy...` | All-based unique lookup with register |
| `\b(update\|remove)(Many\|All)?\s*\(\s*\)` | Parameterless mutation | Context-supplied condition; inspect |
| `\bupdate\s*\(\s*(patch\|changes\|dto\|request)\b` | Bare update with no identity | Identity parameter named differently |
| `\bupdateOne\s*\(\s*(patch\|changes\|dto\|request)\b` | Single update with no identity | Identity parameter named differently |
| `\bfindAll\b|\bAllBy\b` | `All` without an adaptation | Approved All-based register |
| `findMany\w*\([^)]*\)\s*:\s*.*\|\s*null` | Nullable collection | Unrelated union; inspect |
| `\bget(One\|Many)\w*\([^)]*\)\s*:\s*.*\|\s*null` | `get` returning absence | Test helper |
| `BEGIN crud-scope-standardizer` | Managed block marker | Marker inside a code fence |

Use the compiler or AST for return types, interface members, entity-noun repetition, `updateOne` identity, and call-site coverage. Test the analyzer with a positive and a negative fixture.

## 3. Manual review

### R1

- Do CRUD operations use `insert`, `find`, `update`, `remove`?
- Is a synonym a genuinely distinct lifecycle operation?
- Is an upsert hiding insert and update?

### R2

- Do explicit conditions use `By` after scope, in parameter order, without `Or`?
- Is the single-update identity exception used only for the single-update verb (`updateOne` or its profile equivalent) with a required identity?
- Is `updateOne(patch)`, `updateOne()`, `update(patch)`, or `remove()` present?

### R3

- Does a member repeat the enclosing aggregate noun?
- Is a repeated noun a permitted condition on another aggregate?
- Does a cross-aggregate read use a dedicated repository or read model?

### R4

- Does `findOne` return an absence value rather than throwing?
- Does `findMany` return an empty collection rather than null?
- Does `get` throw or return a typed error on absence?
- Does `tryGet` return an explicit result distinct from `find`?
- Is the condition uniqueness-guaranteeing?

### R5

- Is cardinality distinguishable?
- Is `All` absent unless an adaptation is recorded?
- Does `findMany()` return the declared full scope?
- Do bulk updates and removals require a condition and a runtime guard?
- Are `updateAll()`/`removeAll()` parameterless forms absent?
- Is an empty identifier collection a no-op?

### R6

- Is the order verb, scope, condition, suffix?
- Is the transport suffix outside the semantic name?
- Is casing consistent with the language and register?
- Is the repository type name or the word `Repository` absent from members?

### R7

- Are collections non-null?
- Does `get` never return absence?
- Are infrastructure errors distinguishable from absence?
- Are mutation return policies consistent?

### R8–R10

- Is every adaptation or waiver recorded with all required fields?
- If none exists, does the block state that no decisions exist?
- Is the block singular, profile-aligned, and idempotent?
- Is there an audit, a plan, an explicit approval, and validation evidence?

## 4. Labels and acceptance

A supported label is assigned only after validation:

| Label | Required evidence |
| --- | --- |
| Strict | All core rules pass; default profile; no adaptations or waivers |
| Adapted | All core rules pass; configurable choices recorded |
| Partial | Core rule waived with a recorded decision; not compliant |
| Non-compliant | Core rule violated without a waiver |
| Audit-only | No migration or emission; no compliance claim |

A change set is accepted when:

1. every in-scope contract passes R1–R7 or has a recorded waiver;
2. the register is current when adaptations exist; a strict project states none exist;
3. build, tests, and the naming scan pass from a clean checkout, except approved baseline exclusions;
4. contract tests cover absence, collections, updates, removals, and guards;
5. the public API diff contains only approved changes;
6. serialization, reflection, DI, and generated-code checks pass;
7. the block is singular, profile-aligned, and idempotent;
8. the report records evidence for every dimension;
9. the label matches the evidence;
10. every shim and waiver has an owner and review date.

## 5. Severity

| Severity | Examples | Action |
| --- | --- | --- |
| Blocker | Unbounded `removeMany`; upsert under `create`; `get` returning null | Stop; fix or record a waiver |
| High | `findByStatus`; `findOneUserById`; nullable collection | Fix in the current change set |
| Medium | Grammar drift; mixed mutation returns; missing ordering | Fix before next release or record a plan |
| Low | Naming polish; missing example | Fix when the area is next touched |

## 6. Enforcement

At least one mechanism must be configured and tested:

- pre-commit scan;
- pull-request checklist;
- CI naming scan;
- scheduled audit;
- public contract snapshot.

The fixture must include `createUser`, `findByStatus`, `findOneUserById`, `getOneById` returning null, `findMany` returning null, `updateMany()`, `updateOne(patch)`, `updateAll()`, `removeAll()`, `update(patch)`, `remove()`, and a duplicate block. The scan must flag each one.

## 7. Validation report

```markdown
# Repository Contract Validation

## Scope
Project, commit, branch, date; in-scope modules; exclusions.

## Profile
Strict / Adapted / Partial; decision IDs; register path; block target.

## Checks
| Dimension | Method | Result | Evidence |

## Findings
| ID | Rule | Severity | Symbol | Evidence | Status |

## Waivers
| Decision | Rule | Consequence | Approver | Review |

## Label
Supported label; rationale; unresolved findings.

## Sign-off
Reviewer; date; conditions.
```

## 8. Post-migration monitoring

Monitor not-found and validation response rates, list latency after pagination changes, bulk-mutation guard failures, runtime resolution errors after DI changes, new repository methods added without the scan, and drift between the register and the block.

## 9. Production gate

- [ ] Scope and exclusions stated.
- [ ] Every core rule has a verdict with evidence.
- [ ] Every adaptation or waiver recorded; strict states none.
- [ ] Contract tests cover absence, collections, updates, removals, guards.
- [ ] Build and tests pass, except approved baseline exclusions.
- [ ] Naming scan and its fixture pass.
- [ ] API diff approved.
- [ ] Serialization, reflection, DI, generated code checked.
- [ ] Block singular and idempotent.
- [ ] Report complete and signed off.
- [ ] Monitoring and rollback in place.
- [ ] Supported label accurate.
