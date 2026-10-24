# Migration Planning

Status: Normative procedure
Source: [00-rules.md](00-rules.md)
Predecessor: [03-audit.md](03-audit.md)

Planning is Phase B of the migration gate and is read-only. A plan is not an approval. The plan must be specific enough to verify and must state what is not feasible.

## 1. Required sections

Every migration plan contains these six sections, in any order that reads clearly:

1. feasibility;
2. benefits;
3. drawbacks;
4. breaking changes;
5. migration steps;
6. target architecture.

Add risks, rollback, ownership, effort, and an approval request. A plan missing one of the six sections is incomplete.

## 2. Feasibility

State one of:

- **Full:** every in-scope contract can reach the target architecture.
- **Partial:** a named subset can migrate; the remainder is excluded with a reason.
- **Not feasible:** the target cannot be reached without violating a core rule, breaking an unowned public contract, or losing required behavior.

Record preconditions separately from findings. Preconditions may include plan approval, a clean baseline, consumer coordination, a compatibility window, or a data migration. Do not say approval is satisfied before the gate.

## 3. Benefits

Use concrete measures where evidence allows:

| Measure | Before | After |
| --- | --- | --- |
| Canonical CRUD methods | Count | Count |
| Synonym verbs | Count | Count |
| Ambiguous read cardinality | Count | Count |
| Nullable collection returns | Count | Count |
| Unbounded mutation methods | Count | Count |
| Call sites migrated | Count | Count |

Qualitative benefits are acceptable, but they must be tied to a finding or a constraint. Do not claim performance or safety improvements without evidence.

## 4. Drawbacks

Record:

- delivery cost and coordination cost;
- temporary compatibility shims and their removal conditions;
- behavior changes, especially absence-to-error mapping;
- reviewer load and the risk of a mechanical rename;
- any capability that remains unresolved.

## 5. Breaking changes

Classify each surface as none, internal, or public:

| Surface | Questions |
| --- | --- |
| Source | Do in-repo consumers need edits? |
| Binary | Is a compiled artifact distributed? |
| Public API | Are published members renamed or removed? |
| Public behavior | Do status codes, absence results, or errors change? |
| Serialized data | Are method names, payload keys, or event names persisted? |
| Reflection / DI | Is a member resolved by string or convention? |
| Configuration | Does a config key or handler name change? |
| Documentation | Do guides, examples, or generated docs change? |
| Tests | Do external test contracts depend on names? |

A public behavior change requires an explicit condition in the approval. A source-compatible alias must delegate to the target method and preserve semantics.

## 6. Migration steps

Each phase states:

- the action and scope;
- the owner and effort;
- the compatibility strategy;
- the verification command and exit condition;
- the rollback unit;
- whether a new approval is required.

A typical sequence:

1. Record the approval and decision register.
2. Add characterization tests.
3. Introduce the target contract.
4. Add an adapter or compatibility implementation.
5. Migrate services and boundaries.
6. Migrate call sites slice by slice.
7. Deprecate and remove the legacy surface.
8. Add enforcement.

A hard cut is permitted only when the plan justifies it. A deprecation window is required for a public contract unless the owner accepts a major-version break.

## 7. Target architecture

Describe:

- layer boundaries and which layer owns `find`, `get`, and `tryGet`;
- the canonical repository surface with naming examples;
- the declared visibility scope and default ordering;
- the mutation return policy;
- the absence and error model;
- the enforcement mechanism;
- any exclusions and why they remain.

Example target surface:

```text
insertOne
insertMany
findOneById
findMany
findManyByStatus
updateOneById
updateManyByStatus
removeOneById
removeManyByStatus
```

A bare `updateOne(user)` or `updateOne(id, patch)` is permitted when the identity is required; `updateOneById` is preferred. Bulk mutations require a condition and a runtime guard.

In an All-based profile, the target uses `findById`, `findAll`, `findAllByStatus`, `updateById`, and `removeById`; `insertOne` and `insertMany` remain. The register records the mapping.

## 8. Risks and rollback

Each risk records likelihood (1–3), impact (1–3), score, mitigation, trigger, and contingency. A score of 6 or more requires a contingency.

Rollback must be defined per phase:

- the revert unit (commit, adapter, flag, or data restore);
- the verification after revert;
- the owner and escalation path;
- whether a released public change needs a deprecation or rollback release.

A phase that cannot be rolled back safely is not ready for execution.

## 9. Approval gate

The plan ends with an explicit request:

```text
Plan presented. No source modification will occur until the approver records:
- the approval statement;
- the approved scope;
- the approved breaking-change classification;
- any conditions;
- the approver and date.
```

Approval rules:

- "Approve the migration plan" or an equivalent unambiguous statement qualifies.
- "Looks fine", silence, or a request for more detail does not.
- Phase approval authorizes only that phase unless a blanket phase list is approved.
- Conditional approval with unresolved conditions is not approval.
- A material change to scope, target architecture, or breaking-change classification requires a new confirmation.

## 10. Plan checklist

- [ ] All six required sections are present.
- [ ] Feasibility is one of full, partial, or not feasible, with reasons.
- [ ] Benefits and drawbacks are tied to evidence.
- [ ] Every breaking-change surface is classified.
- [ ] Phases have owner, effort, compatibility, verification, and rollback.
- [ ] Target architecture names the layer boundaries and enforcement.
- [ ] Risks have likelihood, impact, score, trigger, and contingency.
- [ ] The approval request is explicit and no approval is assumed.
- [ ] No source was modified.
