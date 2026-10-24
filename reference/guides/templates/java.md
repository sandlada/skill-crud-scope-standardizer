# Java Template

Status: Supporting template
Normative source: [../../core/00-rules.md](../../core/00-rules.md)

## 1. Repository contract

```java
import java.time.Instant;
import java.util.List;
import java.util.Optional;
import java.util.UUID;

public record UserId(UUID value) {
    public UserId {
        if (value == null) throw new IllegalArgumentException("id is required");
    }
}

public enum UserStatus { ACTIVE, SUSPENDED, REMOVED }

public record User(UserId id, String email, UserStatus status, Instant createdAt) {}

public record UserPatch(String email, UserStatus status) {}

public record FindManyOptions(Integer limit, int offset, UserOrderBy orderBy) {
    public static FindManyOptions defaults() {
        return new FindManyOptions(null, 0, UserOrderBy.CREATED_AT_DESC);
    }
}

public enum UserOrderBy { CREATED_AT_DESC, ID_ASC }

public interface UserRepository {
    User insertOne(User user);

    List<User> insertMany(List<User> users);

    Optional<User> findOneById(UserId id);

    List<User> findMany(FindManyOptions options);

    default List<User> findMany() {
        return findMany(FindManyOptions.defaults());
    }

    List<User> findManyByStatus(UserStatus status, FindManyOptions options);

    boolean updateOneById(UserId id, UserPatch patch);

    int updateManyByStatus(UserStatus status, UserPatch patch);

    boolean removeOneById(UserId id);

    int removeManyByStatus(UserStatus status);
}
```

## 2. Service boundary

```java
public final class UserNotFoundException extends RuntimeException {
    private final UserId userId;

    public UserNotFoundException(UserId userId) {
        super("User not found: " + userId.value());
        this.userId = userId;
    }

    public UserId userId() { return userId; }
}

public record LookupResult<T>(boolean found, T value) {
    public static <T> LookupResult<T> found(T value) {
        return new LookupResult<>(true, value);
    }

    public static <T> LookupResult<T> notFound() {
        return new LookupResult<>(false, null);
    }
}

public final class UserService {
    private final UserRepository users;

    public UserService(UserRepository users) { this.users = users; }

    public User getOneById(UserId id) {
        return users.findOneById(id)
            .orElseThrow(() -> new UserNotFoundException(id));
    }

    public LookupResult<User> tryGetOneById(UserId id) {
        return users.findOneById(id)
            .map(LookupResult::found)
            .orElseGet(LookupResult::notFound);
    }
}
```

## 3. Notes

- `Optional<User>` is the `find` contract, not the `get` or `tryGet` contract.
- `findMany` returns an empty list, never `null`.
- `tryGet` returns `LookupResult`, which distinguishes absence from failure.
- `updateOneById` returns whether a record was updated; bulk updates return a count.
- A bare `updateOne(User user)` is permitted only when the required `user.id()` is validated; `updateOneById` is preferred.
- If the project uses an All-based profile, map to `findById`, `findAll`, `findAllByStatus`, `updateById`, and `removeById`; keep `insertOne`/`insertMany`.
- Put `get` and `tryGet` in the service; the repository exposes `find`.
