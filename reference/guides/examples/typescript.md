# TypeScript End-to-End Example

Status: Supporting example
Normative source: [../../core/00-rules.md](../../core/00-rules.md)

This example follows a small user service from audit to validated migration. It uses the default `One`/`Many` profile.

## 1. Initial contract

```ts
export interface UserRepository {
  createUser(input: NewUser): Promise<User>;
  getUserById(id: string): Promise<User | null>;
  getAllUsers(): Promise<User[] | null>;
  findUsersByStatus(status: string): Promise<User[]>;
  deleteUser(id: string): Promise<void>;
}
```

The implementation uses an upsert for `createUser` and a full-table load for `getAllUsers`.

## 2. Audit findings

| ID | Rule | Severity | Evidence | Finding |
| --- | --- | --- | --- | --- |
| F1 | R1, R7 | Blocker | `src/users/user.repository.ts#createUser` | Upsert conflates insert and update. |
| F2 | R1, R7 | High | `src/users/user.repository.ts#deleteUser` | `delete` is not `remove`; `void` hides whether a record existed. |
| F3 | R1, R4 | High | `src/users/user.repository.ts#getUserById` | `get` returns `null`; absence is silent. |
| F4 | R1, R5, R7 | High | `src/users/user.repository.ts#getAllUsers` | `All`, nullable collection, full-table load. |
| F5 | R2, R3, R6 | High | `src/users/user.repository.ts#findUsersByStatus` | Missing scope, repeated `Users`, untyped status. |
| F6 | R5, R7 | Medium | `src/users/user.repository.ts` | No declared visibility scope or ordering. |

Candidate compliance level: Non-compliant.

## 3. Migration plan

### Feasibility

Full, subject to approval. One internal admin console consumes HTTP only; no public SDK, no persisted method names, and no schema change.

### Benefits

| Measure | Before | After |
| --- | --- | --- |
| Canonical CRUD methods | 0 of 5 | 9 of 9 |
| Synonym verbs | 4 | 0 |
| Nullable collections | 1 | 0 |
| Ambiguous read cardinality | 2 | 0 |

### Drawbacks

- 23 call sites in 6 files and 2 test files require edits; estimate 18 engineer-hours.
- A hard cut is acceptable only because the consumer is internal.
- `DELETE /users/:id` changes from unconditional 204 to 404 for a missing user.

### Breaking changes

| Surface | Classification | Detail |
| --- | --- | --- |
| Source | Internal | Repository and service members renamed. |
| Binary | None | No compiled artifact distributed. |
| Public API shape | None | Routes and payload keys unchanged. |
| Public API behavior | Public | Delete returns 404 when absent; admin-console owner accepts the change. |
| Serialized data | None | No column, JSON key, or event field changes. |
| Reflection / DI | None | No runtime lookup by member name. |
| Configuration | None | No new key. |
| Documentation | Internal | Repository guide updated. |
| Tests | Internal | Two test files updated; no external contract. |

### Steps

| Phase | Action | Verification | Rollback |
| --- | --- | --- | --- |
| 1 | Record approval; create register with zero entries; emit strict block. | One block; profile strict. | Restore instruction file. |
| 2 | Add canonical interface and adapter. | `tsc --noEmit`; contract tests. | Revert commit. |
| 3 | Add service `getOneById`, `tryGetOneById`, `findManyByStatus`, `removeOneById`. | Absence and removal tests. | Revert commit. |
| 4 | Migrate 23 call sites. | Route tests; `rg` finds no legacy member. | Revert route commit. |
| 5 | Remove shims; add guards and tests. | Full suite: 18 tests. | Revert commit. |
| 6 | Add CI naming scan. | CI passes clean. | Remove CI step. |

### Target architecture

```ts
export interface UserRepository {
  insertOne(user: User): Promise<User>;
  insertMany(users: readonly User[]): Promise<readonly User[]>;
  findOneById(id: UserId): Promise<User | null>;
  findMany(options?: FindManyOptions): Promise<readonly User[]>;
  findManyByStatus(
    status: UserStatus,
    options?: FindManyOptions,
  ): Promise<readonly User[]>;
  updateOneById(id: UserId, patch: UserPatch): Promise<boolean>;
  updateManyByStatus(status: UserStatus, patch: UserPatch): Promise<number>;
  removeOneById(id: UserId): Promise<boolean>;
  removeManyByStatus(status: UserStatus): Promise<number>;
}
```

`findOneById` returns `null` for absence. `findMany` returns `[]`. `getOneById` lives in the service and throws `UserNotFoundError`; `tryGetOneById` returns a discriminated `Result`. `updateOneById` is preferred; `updateOne(user)` is permitted only with a validated aggregate id.

### Risks

| Risk | L | I | Score | Contingency |
| --- | --- | --- | --- | --- |
| Delete 404 surprises the console | 2 | 3 | 6 | Restore 204 and notify the consumer. |
| A read omits the tenant predicate | 2 | 3 | 6 | Block release, revert, add a two-tenant test. |
| A call site is missed | 1 | 3 | 3 | Revert the slice and rescan dynamic references. |

## 4. Confirmation gate

```text
STOP. No source modification occurs before the approver records:
- "Approve the migration plan as presented."
- Scope: src/users, src/api, test/users.
- Breaking change: DELETE 404 accepted.
- Approver and date.
```

A dry-run approval is recorded below for illustration only.

| Field | Value |
| --- | --- |
| Statement | "Approve the migration plan as presented." |
| Approver | Example operator |
| Date | 2026-10-24 |
| Conditions | 404 behavior accepted; no schema change. |

## 5. Validation

| Check | Result |
| --- | --- |
| Naming scan | 0 legacy members; 9 canonical methods |
| `findMany` nullability | Non-null; empty array |
| `get` absence | Throws `UserNotFoundError` |
| `tryGet` | Returns explicit `Result` |
| Identity update | `updateOneById` preferred; `updateOne(user)` validated |
| Bulk guards | Reject empty condition; empty id collection is a no-op |
| Managed block | One block; strict profile |
| Label | Candidate strict; supported after full test evidence |

## 6. Lessons

- The audit must distinguish `get` as a read alias from `get` as a service not-found contract.
- The identity-bearing `updateOne` is not a conditionless mutation; it is bounded by the aggregate id.
- A nullable collection is a contract defect even when callers currently handle it.
- The confirmation gate prevents a mechanical rename from becoming an unapproved behavior change.
