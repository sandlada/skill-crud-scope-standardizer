# Python Template

Status: Supporting template
Normative source: [../../core/00-rules.md](../../core/00-rules.md)

## 1. Repository contract

```python
from dataclasses import dataclass
from enum import Enum
from typing import Protocol, Sequence
from uuid import UUID


class UserStatus(Enum):
    ACTIVE = "active"
    SUSPENDED = "suspended"
    REMOVED = "removed"


@dataclass(frozen=True, slots=True)
class UserId:
    value: UUID

    def __post_init__(self) -> None:
        if self.value.int == 0:
            raise ValueError("id is required")


@dataclass(frozen=True, slots=True)
class User:
    id: UserId
    email: str
    status: UserStatus
    created_at: object


@dataclass(frozen=True, slots=True)
class UserPatch:
    email: str | None = None
    status: UserStatus | None = None


@dataclass(frozen=True, slots=True)
class FindManyOptions:
    limit: int | None = None
    offset: int = 0
    order_by: tuple[tuple[str, bool], ...] = ()


class UserRepository(Protocol):
    async def insert_one(self, user: User) -> User: ...

    async def insert_many(self, users: Sequence[User]) -> Sequence[User]: ...

    async def find_one_by_id(self, user_id: UserId) -> User | None: ...

    async def find_many(
        self, options: FindManyOptions | None = None
    ) -> Sequence[User]: ...

    async def find_many_by_status(
        self, status: UserStatus, options: FindManyOptions | None = None
    ) -> Sequence[User]: ...

    async def update_one_by_id(
        self, user_id: UserId, patch: UserPatch
    ) -> bool: ...

    async def update_many_by_status(
        self, status: UserStatus, patch: UserPatch
    ) -> int: ...

    async def remove_one_by_id(self, user_id: UserId) -> bool: ...

    async def remove_many_by_status(self, status: UserStatus) -> int: ...
```

## 2. Service boundary

```python
from dataclasses import dataclass
from typing import Generic, TypeVar

T = TypeVar("T")


class UserNotFoundError(Exception):
    def __init__(self, user_id: UserId) -> None:
        super().__init__(f"User not found: {user_id.value}")
        self.user_id = user_id


@dataclass(frozen=True, slots=True)
class LookupResult(Generic[T]):
    found: bool
    value: T | None = None


class UserService:
    def __init__(self, users: UserRepository) -> None:
        self._users = users

    async def get_one_by_id(self, user_id: UserId) -> User:
        user = await self._users.find_one_by_id(user_id)
        if user is None:
            raise UserNotFoundError(user_id)
        return user

    async def try_get_one_by_id(self, user_id: UserId) -> LookupResult[User]:
        user = await self._users.find_one_by_id(user_id)
        if user is None:
            return LookupResult(found=False)
        return LookupResult(found=True, value=user)
```

## 3. Notes

- `None` is the `find` contract; `LookupResult` is the explicit `tryGet` contract.
- `find_many` returns an empty sequence, never `None`.
- `UserNotFoundError` carries the lookup key and no sensitive data.
- A bare `update_one(user)` is permitted only when `user.id` is required and validated; `update_one_by_id` is preferred.
- If the project uses an All-based profile, map to `find_by_id`, `find_all`, `find_all_by_status`, `update_by_id`, and `remove_by_id`; keep `insert_one`/`insert_many`.
- Put `get` and `try_get` in the service; the repository exposes `find`.
