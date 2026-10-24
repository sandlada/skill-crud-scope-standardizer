# C# Template

Status: Supporting template
Normative source: [../../core/00-rules.md](../../core/00-rules.md)

## 1. Repository contract

```csharp
using System;
using System.Collections.Generic;
using System.Threading;
using System.Threading.Tasks;

namespace App.Users;

public readonly record struct UserId(Guid Value)
{
    public static UserId New() => new(Guid.NewGuid());
}

public enum UserStatus { Active, Suspended, Removed }

public sealed record User(
    UserId Id,
    string Email,
    UserStatus Status,
    DateTimeOffset CreatedAt);

public sealed record UserPatch(string? Email, UserStatus? Status);

public sealed record FindManyOptions(
    int? Limit = null,
    int Offset = 0,
    UserOrderBy OrderBy = UserOrderBy.CreatedAtDescending);

public enum UserOrderBy { CreatedAtDescending, IdAscending }

public interface IUserRepository
{
    Task<User> InsertOneAsync(User user, CancellationToken cancellationToken = default);

    Task<IReadOnlyList<User>> InsertManyAsync(
        IReadOnlyList<User> users,
        CancellationToken cancellationToken = default);

    Task<User?> FindOneByIdAsync(
        UserId id,
        CancellationToken cancellationToken = default);

    Task<IReadOnlyList<User>> FindManyAsync(
        FindManyOptions? options = null,
        CancellationToken cancellationToken = default);

    Task<IReadOnlyList<User>> FindManyByStatusAsync(
        UserStatus status,
        FindManyOptions? options = null,
        CancellationToken cancellationToken = default);

    Task<bool> UpdateOneByIdAsync(
        UserId id,
        UserPatch patch,
        CancellationToken cancellationToken = default);

    Task<int> UpdateManyByStatusAsync(
        UserStatus status,
        UserPatch patch,
        CancellationToken cancellationToken = default);

    Task<bool> RemoveOneByIdAsync(
        UserId id,
        CancellationToken cancellationToken = default);

    Task<int> RemoveManyByStatusAsync(
        UserStatus status,
        CancellationToken cancellationToken = default);
}
```

## 2. Service boundary

```csharp
public sealed class UserNotFoundException : Exception
{
    public UserId UserId { get; }

    public UserNotFoundException(UserId userId)
        : base($"User not found: {userId.Value}")
    {
        UserId = userId;
    }
}

public readonly record struct LookupResult<T>(bool Found, T? Value)
{
    public static LookupResult<T> FoundValue(T value) => new(true, value);
    public static LookupResult<T> NotFound() => new(false, default);
}

public sealed class UserService
{
    private readonly IUserRepository _users;

    public UserService(IUserRepository users) => _users = users;

    public async Task<User> GetOneByIdAsync(
        UserId id,
        CancellationToken cancellationToken = default)
    {
        var user = await _users.FindOneByIdAsync(id, cancellationToken);
        return user ?? throw new UserNotFoundException(id);
    }

    public async Task<LookupResult<User>> TryGetOneByIdAsync(
        UserId id,
        CancellationToken cancellationToken = default)
    {
        var user = await _users.FindOneByIdAsync(id, cancellationToken);
        return user is null
            ? LookupResult<User>.NotFound()
            : LookupResult<User>.FoundValue(user);
    }
}
```

## 3. Notes

- Enable nullable reference types so `User?` is meaningful.
- `FindManyAsync` returns an empty `IReadOnlyList<User>`, never `null`.
- `TryGetOneByIdAsync` returns a distinct lookup result, not `User?`.
- A bare `UpdateOneAsync(User user)` is permitted only when `user.Id` is required and validated; `UpdateOneByIdAsync` is preferred.
- `RemoveOneByIdAsync` returns `bool`; `RemoveManyByStatusAsync` returns a count.
- If the project uses an All-based profile, map to `FindByIdAsync`, `FindAllAsync`, `FindAllByStatusAsync`, `UpdateByIdAsync`, and `RemoveByIdAsync`; keep `InsertOneAsync`/`InsertManyAsync`.
- Put `Get` and `TryGet` in the service; the repository exposes `Find`.
