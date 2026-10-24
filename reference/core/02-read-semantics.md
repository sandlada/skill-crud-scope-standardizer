# Read Semantics

Status: Normative application guide
Source: [00-rules.md](00-rules.md)

This file defines the `find`, `tryGet`, and `get` families. The priority is:

```text
find > tryGet > get
```

Prefer `find`. Introduce `tryGet` when the caller must branch on absence without an exception. Introduce `get` only when the surrounding contract states that the record must exist.

## 1. The three outcomes

| Outcome | Meaning | Default representation |
| --- | --- | --- |
| Found | A matching record exists | The record |
| Absent | No matching record exists; this is normal | Absence value or explicit lookup result |
| Failed | Persistence or infrastructure failed | Typed failure, rejected promise, or exception |

Absence and failure MUST remain distinguishable. Never convert a failure into an empty collection or a not-found result.

## 2. Family contracts

### `find`

- `findOneBy...` returns the record or an absence value.
- `findMany` and `findManyBy...` return a non-null collection, empty when there are no matches.
- Absence is not an exception.
- A `findOne` condition must guarantee uniqueness.

```ts
findOneById(id): Promise<User | null>;
findMany(): Promise<readonly User[]>;
findManyByStatus(status: UserStatus): Promise<readonly User[]>;
```

### `tryGet`

- Returns an explicit success-or-absence result.
- Does not throw for absence.
- May propagate infrastructure failures.
- The result type must be distinct from a plain nullable value, otherwise it adds no information over `find`.

Examples of conforming shapes: a discriminated union, a dedicated `LookupResult<T>`, a result type, or a tuple such as `(T, bool)`.

### `get`

- Returns the record when it exists.
- Raises or returns a typed not-found error when it does not.
- Never returns `null`, `undefined`, `None`, or an empty optional.
- Identifies the aggregate and lookup key in the error.

## 3. Layer placement

The default repository surface exposes `find` only.

| Layer | Responsibility | Typical family |
| --- | --- | --- |
| Repository | Query persistence and return absence as data | `find` |
| Service/application | Apply use-case rules and translate absence | `get`, `tryGet` |
| Boundary/transport | Map service outcomes to protocol responses | Consumes service results |

A repository may expose `get` or `tryGet` only as a recorded R8 opt-in, and the methods must still obey the semantics above. `get` is not a synonym for `find`.

## 4. Return-type matrix

The table is representative. A project may use its own result type if the absence and failure channels remain distinct.

| Language | `find` | `findMany` | `get` | `tryGet` |
| --- | --- | --- | --- | --- |
| TypeScript | `Promise<T \| null>` | `Promise<readonly T[]>` | `Promise<T>` rejecting with typed error | `Promise<Result<T, NotFoundError>>` |
| C# | `Task<T?>` | `Task<IReadOnlyList<T>>` | `Task<T>` throwing typed exception | `Task<LookupResult<T>>` or `bool TryGet(out T)` |
| Java | `Optional<T>` | `List<T>` | `T` throwing typed exception | `LookupResult<T>` |
| Kotlin | `T?` | `List<T>` | `T` throwing typed exception | `Lookup<T>` |
| Python | `T \| None` | `Sequence[T]` | `T` raising typed exception | `LookupResult[T]` or `(T, bool)` |
| Go | `(*T, error)` | `([]T, error)` | `(T, error)` with typed not-found error | `(LookupResult[T], error)` |
| Rust | `Result<Option<T>, E>` | `Result<Vec<T>, E>` | `Result<T, E>` with not-found variant | `Result<Lookup<T>, E>` |

## 5. Uniqueness

A `findOne` or `getOne` condition must match at most one record. Use an identifier, a complete unique key, or a documented unique domain key.

If duplicates are possible:

- use `findManyBy...` with an explicit limit and documented selection order; or
- add a uniqueness constraint through a separate approved change.

Silently returning the first row converts a data defect into a permanent contract.

## 6. Error mapping

| Condition | Repository | Service | Boundary |
| --- | --- | --- | --- |
| No match, normal read | Absence value | Preserve absence or map to use-case decision | 200 with empty result, or 404 where appropriate |
| No match, required record | Absence value | Typed not-found error | 404 |
| Invalid input | Validation error | Validation error | 400 or 422 |
| Conflict | Conflict error | Conflict error | 409 |
| Infrastructure failure | Repository failure | Propagated or translated | 500 or 503 |

Do not leak raw database errors, credentials, tokens, or internal connection details.

## 7. Anti-patterns

| Incorrect | Correct | Reason |
| --- | --- | --- |
| `getUserById` returning `null` | `findOneById` or service `getOneById` throwing | `get` must not return absence |
| `findMany` returning `null` | `findMany` returning `[]` | Collections are never null |
| `tryGet` returning `T \| None` | `tryGet` returning an explicit lookup result | Must be distinct from `find` |
| `getOneById` returning an empty optional | `getOneById` throwing a typed error | `get` treats absence as an error |
| Empty collection for a failed query | Typed failure | Failure is not absence |
| `findOneByStatus` with many matches | `findManyByStatus` with limit and order | `findOne` requires uniqueness |

## 8. Decision checklist

- Is absence a normal outcome or an error for this caller?
- Does the method name match the chosen family?
- Does the return type make absence observable?
- Is failure distinguishable from absence?
- Is the condition uniqueness-guaranteeing?
- Is `get` or `tryGet` at the correct layer?
- Is the error typed and free of sensitive details?
