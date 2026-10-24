# Migration Execution

Status: Normative procedure
Source: [00-rules.md](00-rules.md)
Predecessors: [03-audit.md](03-audit.md), [04-plan.md](04-plan.md)

Execution is Phase D of the migration gate. It begins only after an explicit approval of the plan and its breaking-change classification. Execution is not a rename sweep; it is a controlled change program with a baseline, compatibility strategy, verification, and rollback.

## 1. Preconditions

| Precondition | Evidence |
| --- | --- |
| Plan approved | Approval statement, approver, date, scope, conditions |
| Scope frozen | Modules, packages, contracts, call sites, tests, exclusions |
| Register current | Every adaptation or waiver recorded |
| Baseline recorded | Commit, build result, test count, API snapshot, known failures |
| Baseline green | Build and tests pass, or approved exclusions are recorded |
| Rollback defined | Revert unit and data-safety assessment per phase |
| Ownership | Accountable owner and reviewer per slice |
| Tooling pinned | Formatter, compiler, test runner, naming scan |

If the baseline is not green, stop unless the approved plan records the known failures and excludes them from acceptance. An unrecorded failure is a stop condition. If the plan was approved against a different commit, revalidate it.

## 2. Freeze and branch

- Announce a short freeze for target modules.
- Create a dedicated branch from the recorded baseline.
- Keep formatting-only changes separate from semantic renames.
- Commit each phase or slice with its rule IDs and verification evidence.
- Do not upgrade the compiler, test runner, or schema unless the plan includes it.

## 3. Characterization tests

Before changing a contract, add or identify tests for:

- found, absent, and invalid-identifier reads;
- empty and multiple-result collections;
- update success, missing target, invalid patch, identity mismatch;
- bulk update and removal with matches, no matches, and rejected missing conditions;
- empty identifier collections as no-ops;
- not-found, validation, conflict, and infrastructure failure mapping;
- serialization and persisted names where applicable.

A migration without executable verification is not approved for production.

## 4. Phase sequence

### Phase 1: target contract

Add the canonical interface alongside the legacy contract. Use `One`/`Many` or the approved adapted vocabulary. Apply the return and error contracts. Put `get` and `tryGet` in the service or boundary layer unless an R8 opt-in exists. Document visibility scope, ordering, uniqueness, and empty-collection behavior.

### Phase 2: adapter

Implement the target contract by delegating to existing persistence logic. Do not rewrite queries during the naming phase. If both contracts must coexist, use one adapter that delegates to the same core. Implement bulk guards and identity validation before migrating call sites. A bare single update is permitted only when a required parameter carries the aggregate root's unique id; `updateOneById` (or the profile equivalent `updateById`) is preferred, and `updateOne(patch)` or `updateOne()` is invalid. Bulk updates and removals still require a `By` condition and a runtime guard. Mark the adapter with a removal condition.

### Phase 3: service and boundary

Replace repository `get` aliases with `find` plus an explicit absence decision. Introduce `getOneById` only where absence is an error; introduce `tryGetOneById` only for an explicit non-throwing result. Preserve routes, payload keys, and status codes unless the plan classifies a change as public and accepts it.

### Phase 4: call sites

Migrate by module, feature, or bounded context. For each slice:

1. replace legacy references;
2. update imports and DI registrations;
3. update tests that assert names;
4. run the slice verification;
5. commit with evidence.

Use the compiler as the primary discovery tool. Search dynamic references, reflection, configuration, serialized names, and generated clients.

### Phase 5: deprecate

Mark legacy members deprecated with the replacement and removal release. Keep behavior unchanged during the window. Do not add new legacy members. An internal package may skip the window if the plan approves a hard cut.

### Phase 6: remove

Remove legacy members and adapters only when no in-scope call site uses them, the API diff contains only approved removals, serialized/reflection checks pass, and the register records the removal. Removal is a separate commit.

### Phase 7: enforce

Add the approved naming scan, analyzer, architecture test, or review checklist. Test the check against a known violation. A check that has never failed is not known to work.

## 5. Compatibility

- **Public API:** classify source, binary, and behavior compatibility. Keep aliases only for the approved window. Publish a migration note. Do not remove a public member in a patch release.
- **Serialized data:** audit job payloads, event names, audit logs, dynamic SQL dispatch, configuration, and generated clients. If a persisted name must remain, version the data contract and map it at the boundary.
- **Reflection/DI:** update string-based lookup, decorators, proxies, and registrations in the same slice. Add a container-resolution smoke test.
- **ORM/generated code:** do not rename entities or columns for this rule. Change the generator or wrap generated code in an adapter.
- **Framework methods:** document as framework surface; do not rename.

## 6. Per-slice verification

| Check | Pass condition |
| --- | --- |
| Format | No diff |
| Compile/type check | Zero errors |
| Unit and contract tests | All pass; no new skips |
| Naming scan | No legacy member in the slice |
| Dynamic references | Updated or explicitly exempted |
| API diff | Only approved changes |
| Serialization | No unapproved change |
| Performance smoke | No unexplained regression |
| Emission, if in scope | Exactly one block; profile matches register |

A slice is complete when its evidence is recorded, not when it compiles.

## 7. Rollback and incidents

Rollback unit is the phase commit, not the whole migration. Revert the phase, re-run baseline verification, and confirm no data or serialized change needs separate rollback.

Stop and escalate when:

- a public change was not classified;
- a test failure is unexplained;
- runtime resolution fails after a rename;
- duplicate data violates a `findOne` uniqueness assumption;
- a bulk guard fails or an unbounded statement is observed;
- a serialized name or external consumer breaks;
- the tree diverges materially from the approved baseline.

Record the trigger, affected phase, revert commit, verification, and resume/revise/abandon decision. An abandoned migration is incomplete, not compliant.

## 8. Change control and completion

A new confirmation is required for a scope change, a different cardinality profile or register, a changed breaking-change classification, a different compatibility strategy or deprecation window, a different target architecture or error model, or a new public behavior change.

The migration is complete when:

1. in-scope contracts use the adopted vocabulary;
2. call sites compile against the target surface;
3. legacy members and adapters are removed or scheduled in the register;
4. the full build and tests pass, except approved baseline exclusions;
5. the naming scan passes;
6. the API diff contains only approved changes;
7. serialization, reflection, DI, and generated-code checks pass;
8. the managed block is singular and profile-aligned;
9. the validation report records evidence and the supported label;
10. the approval and register reflect the delivered state.

## 9. Prohibited actions

- Editing source before approval.
- Renaming framework methods.
- A mechanical rename without characterization tests.
- Introducing ambiguous forms as temporary convenience.
- Leaving an unbounded bulk mutation because callers "always pass a condition".
- Preserving an upsert under a create or save name.
- Hiding a failure as an empty collection or not-found result.
- Keeping a shim without an owner and removal condition.
- Claiming completion from a compiler-green build alone.
- Describing a partial profile as compliant.
