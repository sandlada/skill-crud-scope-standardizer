# Project Audit

Status: Normative procedure
Source: [00-rules.md](00-rules.md)

The audit is Phase A of the migration gate. It is read-only. It produces evidence, findings, and a candidate compliance level; it does not modify source and does not approve a migration.

## 1. Bound the scope

Before reading code, state:

- the target modules, packages, and repositories;
- the aggregate-scoped contracts in scope;
- the languages and casing conventions;
- consumers, call sites, tests, and build constraints;
- excluded surfaces and the reason for each exclusion;
- the commit or tree state being audited.

Out of scope by default: ORM entities, schema objects, migration files, routes, JSON keys, framework lifecycle methods, and private helpers outside a contract.

## 2. Read the architecture

Record evidence as file and symbol references:

| Area | What to inspect |
| --- | --- |
| Contracts | Repository, gateway, port, and service interfaces or traits |
| Implementations | Query construction, mapping, transaction behavior, error translation |
| Call sites | Services, handlers, jobs, generated clients, dynamic dispatch |
| Tests | Contract tests, absence cases, bulk guards, ordering, serialization |
| Errors | Not-found, validation, conflict, infrastructure failure |
| Constraints | Public API, binary compatibility, persisted names, reflection, DI |
| Governance | Existing decision register, instruction blocks, CI checks |

Use the compiler or type checker as the primary call-site discovery mechanism. Supplement with repository-wide search for dynamic references, configuration, and documentation. Do not infer call sites from names alone.

## 3. Per-rule review

| Rule | Question | Failure signal |
| --- | --- | --- |
| R1 | Do CRUD operations use exactly `insert`, `find`, `update`, `remove`? | `create`, `save`, `get`, `list`, `delete`, `destroy`, or another synonym |
| R2 | Do explicit conditions use `By<Condition>` after scope? | Positional conditions, non-identity implicit conditions, column names, `Or` |
| R3 | Does any member repeat the aggregate noun? | `findOneUserById`, `findManyUsersByStatus` |
| R4 | Are `find`, `tryGet`, and `get` distinct and correctly placed? | `get` returning absence; `find` throwing not-found; nullable collection |
| R5 | Is cardinality distinguishable and are mutations bounded? | `findByStatus`, `updateMany()`, `removeMany()`, `updateOne(patch)` |
| R6 | Does the grammar follow verb, scope, condition, suffix? | Transport suffix inside the name; illegal plural; `Or` |
| R7 | Do return and error contracts make absence observable? | Nullable collection; `void` removal; raw database error |
| R8 | Is every adaptation or waiver recorded? | Unrecorded deviation; invalid register entry |
| R9 | Is an existing managed block singular and profile-aligned? | Missing, duplicate, or conflicting block |
| R10 | Is there evidence of audit, plan, approval, execution, validation? | Direct edits without approval; stale approval |

A single-aggregate `updateOne(user)` or `updateOne(id, patch)` is permitted when the identity is a required parameter. `updateOneById` is preferred. `updateOne(patch)` and `updateOne()` are findings.

An All-based profile is an adaptation, not a core waiver, when cardinality remains distinguishable and the parameterless plural read returns the full scope. `insertOne` and `insertMany` remain valid.

## 4. Evidence standard

Every finding must cite:

- a file and symbol;
- the rule ID;
- the observed behavior;
- the impact and severity;
- the suggested direction, without editing.

A finding without a symbol is an observation, not a finding. A rule verdict is one of `pass`, `fail`, `not applicable`, or `waived`. A pass must cover every in-scope contract, not a sample.

## 5. Severity and risk

| Severity | Meaning |
| --- | --- |
| Blocker (S1) | Data loss, unbounded destructive operation, unsafe absence semantics, or a public contract break |
| High (S2) | Ambiguous cardinality, repeated aggregate noun, inconsistent mutation policy, or an unrecorded adaptation |
| Medium (S3) | Grammar drift, inconsistent returns, missing ordering, or weak documentation |
| Low (S4) | Naming polish or a non-blocking documentation gap |

Risk score = severity weight × exposure weight. Exposure is low (1), medium (2), or high (3). Scores 1–2 are low, 3–4 moderate, 5–8 high, and 9–12 critical. A score of 5 or more requires a contingency, not only a mitigation.

## 6. Candidate compliance level

| Condition | Candidate level |
| --- | --- |
| Audit incomplete or evidence insufficient | Audit-only |
| Core rule fails without a valid waiver | Non-compliant |
| Core rule fails with a valid waiver | Partial |
| All core rules pass, with a recorded configurable adaptation | Adapted |
| All core rules pass, with no adaptation or waiver | Strict |

An audit reports a **candidate** level, not a supported compliance label. The supported label is assigned by validation. A read-only audit not followed by validation reports "Audited only; compliance not asserted" unless the requester explicitly accepts the candidate as the audit conclusion; even then, the report must say "candidate".

## 7. Audit report template

```markdown
# Repository Contract Audit

Audit ID:
System and repository:
Baseline commit and tree state:
Audit date and auditor:
Mode: read-only audit (R10 Phase A)
Scope:
Exclusions:

## 1. Executive assessment
Candidate compliance level:
Deciding rule failures:
Evidence gaps:
Supported label: not asserted by this audit

## 2. Inventory
| Contract | Aggregate | Declaration | Methods | Implementation | Consumers |

## 3. Per-rule verdicts
| Rule | Verdict | Severity | Exposure | Risk | Evidence | Summary |

## 4. Findings
| ID | Rule | Severity | Symbol | Evidence | Impact | Direction |

## 5. Constraints and open questions
Build and release constraints, compatibility obligations, ownership, freeze windows.

## 6. Evidence index
| ID | File | Symbol or location | Observation |
```

## 8. Audit checklist

- [ ] Scope and exclusions stated.
- [ ] Contracts, implementations, call sites, and tests inspected.
- [ ] Every finding has a file and symbol.
- [ ] R1–R10 have verdicts or evidence gaps.
- [ ] `updateOne` identity cases were inspected, not flagged mechanically.
- [ ] All-based adaptations were checked against the register.
- [ ] Candidate level follows the verdict table.
- [ ] No source was modified.
- [ ] The report states that the supported label is not asserted.
