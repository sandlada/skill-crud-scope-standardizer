# Kotlin Template

Status: Supporting template
Normative source: [../../core/00-rules.md](../../core/00-rules.md)

## 1. Repository contract

```kotlin
import java.time.Instant
import java.util.UUID

@JvmInline
value class UserId(val value: UUID) {
    init { require(value != UUID(0, 0)) { "id is required" } }
}

enum class UserStatus { ACTIVE, SUSPENDED, REMOVED }

data class User(
    val id: UserId,
    val email: String,
    val status: UserStatus,
    val createdAt: Instant,
)

data class UserPatch(val email: String? = null, val status: UserStatus? = null)

data class FindManyOptions(
    val limit: Int? = null,
    val offset: Int = 0,
    val orderBy: UserOrderBy = UserOrderBy.CREATED_AT_DESC,
)

enum class UserOrderBy { CREATED_AT_DESC, ID_ASC }

interface UserRepository {
    suspend fun insertOne(user: User): User
    suspend fun insertMany(users: List<User>): List<User>
    suspend fun findOneById(id: UserId): User?
    suspend fun findMany(options: FindManyOptions = FindManyOptions()): List<User>
    suspend fun findManyByStatus(
        status: UserStatus,
        options: FindManyOptions = FindManyOptions(),
    ): List<User>
    suspend fun updateOneById(id: UserId, patch: UserPatch): Boolean
    suspend fun updateManyByStatus(status: UserStatus, patch: UserPatch): Int
    suspend fun removeOneById(id: UserId): Boolean
    suspend fun removeManyByStatus(status: UserStatus): Int
}
```

## 2. Service boundary

```kotlin
class UserNotFoundException(val userId: UserId) :
    RuntimeException("User not found: $userId")

sealed interface Lookup<out T> {
    data class Found<T>(val value: T) : Lookup<T>
    data object NotFound : Lookup<Nothing>
}

class UserService(private val users: UserRepository) {
    suspend fun getOneById(id: UserId): User =
        users.findOneById(id) ?: throw UserNotFoundException(id)

    suspend fun tryGetOneById(id: UserId): Lookup<User> =
        users.findOneById(id)?.let { Lookup.Found(it) } ?: Lookup.NotFound
}
```

## 3. Notes

- `T?` is the `find` contract; `Lookup` is the explicit `tryGet` contract.
- `findMany` returns `emptyList()`, never `null`.
- `Result<T>` represents failure, not ordinary absence; do not use `Result.failure` for a missing record in `tryGet`.
- `suspend` is the asynchronous convention and carries no extra suffix.
- A bare `updateOne(user: User)` is permitted only when `user.id` is required and validated; `updateOneById` is preferred.
- If the project uses an All-based profile, map to `findById`, `findAll`, `findAllByStatus`, `updateById`, and `removeById`; keep `insertOne`/`insertMany`.
- Put `get` and `tryGet` in the service; the repository exposes `find`.
