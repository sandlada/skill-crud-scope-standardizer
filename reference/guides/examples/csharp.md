# C# End-to-End Example

Status: Supporting example
Normative source: [../../core/00-rules.md](../../core/00-rules.md)

This example tests the identity-bearing `updateOne` rule in a C# repository.

## 1. Initial contract

```csharp
public interface ICustomerRepository
{
    Task<Customer> AddAsync(Customer customer, CancellationToken ct = default);
    Task<Customer?> GetByIdAsync(Guid id, CancellationToken ct = default);
    Task<List<Customer>?> GetAllAsync(CancellationToken ct = default);
    Task<List<Customer>> ListActiveAsync(CancellationToken ct = default);
    Task UpdateOneAsync(Customer customer, CancellationToken ct = default);
    Task DeleteAsync(Guid id, CancellationToken ct = default);
}
```

## 2. Audit

| ID | Rule | Severity | Evidence | Finding |
| --- | --- | --- | --- | --- |
| F1 | R1 | High | `ICustomerRepository.AddAsync` | `Add` is not `insert`. |
| F2 | R1, R4 | High | `ICustomerRepository.GetByIdAsync` | `get` is a find alias and returns `null`. |
| F3 | R1, R5, R7 | High | `ICustomerRepository.GetAllAsync` | `All`, nullable collection. |
| F4 | R1, R2, R6 | High | `ICustomerRepository.ListActiveAsync` | `list` is not `find`; no scope or `By`. |
| F5 | R1, R6 | High | `ICustomerRepository.DeleteAsync` | `delete` is not `remove`; no affected result. |
| F6 | R2, R5 | Low | `ICustomerRepository.UpdateOneAsync(Customer)` | Identity-bearing update is permitted; `UpdateOneByIdAsync` is preferred. |

Candidate level: Non-compliant.

## 3. Plan

Feasibility: full, subject to approval. The interface is internal and has no consumers.

Benefits: remove four synonym verbs, make read cardinality explicit, remove the nullable collection, and return affected results.

Drawbacks: a hard cut changes the interface; no compatibility shim exists.

Breaking changes: source only; internal; no persisted names, binary, routes, or schema.

Steps:

1. Record approval and a zero-entry register.
2. Replace the interface with the canonical surface.
3. Update call sites.
4. Add contract tests for absence, empty collections, updates, and removals.
5. Add the naming scan.

Target:

```csharp
public interface ICustomerRepository
{
    Task<Customer> InsertOneAsync(
        Customer customer,
        CancellationToken cancellationToken = default);

    Task<IReadOnlyList<Customer>> FindManyAsync(
        CancellationToken cancellationToken = default);

    Task<Customer?> FindOneByIdAsync(
        Guid id,
        CancellationToken cancellationToken = default);

    Task<IReadOnlyList<Customer>> FindManyByActiveAsync(
        bool active,
        CancellationToken cancellationToken = default);

    Task<bool> UpdateOneByIdAsync(
        Guid id,
        CustomerPatch patch,
        CancellationToken cancellationToken = default);

    Task<bool> RemoveOneByIdAsync(
        Guid id,
        CancellationToken cancellationToken = default);
}
```

`UpdateOneAsync(Customer customer)` was already legal under R2 because `customer.Id` is the required identity. The target prefers `UpdateOneByIdAsync` for visibility. If the project keeps `UpdateOneAsync(Customer)`, it must reject a missing or `Guid.Empty` id.

## 4. Gate

```text
STOP. Approval required before editing source.
Approve the plan, the internal breaking change, and the target surface.
```

## 5. Validation

- Legacy scan: `AddAsync`, `GetByIdAsync`, `GetAllAsync`, `ListActiveAsync`, `DeleteAsync` → 0.
- Canonical scan: `InsertOneAsync`, `FindOneByIdAsync`, `FindManyAsync`, `FindManyByActiveAsync`, `UpdateOneByIdAsync`, `RemoveOneByIdAsync`.
- `FindManyAsync` returns an empty list, not null.
- `FindOneByIdAsync` returns `null` for absence.
- `UpdateOneByIdAsync` and `RemoveOneByIdAsync` reject `Guid.Empty`.
- Candidate strict; supported after full test evidence.

## 6. Lesson

An auditor must inspect the `UpdateOneAsync` signature before flagging it. A bare single update is allowed only when the aggregate identity is required and validated; a patch-only update is a defect.
