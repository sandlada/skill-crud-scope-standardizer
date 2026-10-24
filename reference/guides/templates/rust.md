# Rust Template

Status: Supporting template
Normative source: [../../core/00-rules.md](../../core/00-rules.md)

## 1. Repository contract

```rust
use async_trait::async_trait;
use std::time::SystemTime;
use uuid::Uuid;

#[derive(Clone, Copy, Debug, PartialEq, Eq, Hash)]
pub struct UserId(pub Uuid);

#[derive(Clone, Copy, Debug, PartialEq, Eq)]
pub enum UserStatus { Active, Suspended, Removed }

#[derive(Clone, Debug)]
pub struct User {
    pub id: UserId,
    pub email: String,
    pub status: UserStatus,
    pub created_at: SystemTime,
}

#[derive(Clone, Debug, Default)]
pub struct UserPatch {
    pub email: Option<String>,
    pub status: Option<UserStatus>,
}

#[derive(Clone, Debug, Default)]
pub struct FindManyOptions {
    pub limit: Option<u32>,
    pub offset: u32,
    pub order_by: UserOrderBy,
}

#[derive(Clone, Copy, Debug, Default)]
pub enum UserOrderBy {
    #[default]
    CreatedAtDesc,
    IdAsc,
}

#[derive(Debug, thiserror::Error)]
pub enum RepositoryError {
    #[error("storage is unavailable")]
    Unavailable,
    #[error("write conflict")]
    Conflict,
    #[error("stored data is invalid")]
    InvalidData,
}

#[async_trait]
pub trait UserRepository: Send + Sync {
    async fn insert_one(&self, user: User) -> Result<User, RepositoryError>;
    async fn insert_many(&self, users: Vec<User>) -> Result<Vec<User>, RepositoryError>;
    async fn find_one_by_id(&self, id: UserId) -> Result<Option<User>, RepositoryError>;
    async fn find_many(&self, options: FindManyOptions) -> Result<Vec<User>, RepositoryError>;
    async fn find_many_by_status(
        &self,
        status: UserStatus,
        options: FindManyOptions,
    ) -> Result<Vec<User>, RepositoryError>;
    async fn update_one_by_id(
        &self,
        id: UserId,
        patch: UserPatch,
    ) -> Result<bool, RepositoryError>;
    async fn update_many_by_status(
        &self,
        status: UserStatus,
        patch: UserPatch,
    ) -> Result<u64, RepositoryError>;
    async fn remove_one_by_id(&self, id: UserId) -> Result<bool, RepositoryError>;
    async fn remove_many_by_status(
        &self,
        status: UserStatus,
    ) -> Result<u64, RepositoryError>;
}
```

## 2. Service boundary

```rust
use std::sync::Arc;

#[derive(Debug, thiserror::Error)]
#[error("user not found: {user_id:?}")]
pub struct UserNotFoundError { pub user_id: UserId }

#[derive(Debug, thiserror::Error)]
pub enum UserServiceError {
    #[error(transparent)]
    Repository(#[from] RepositoryError),
    #[error(transparent)]
    NotFound(#[from] UserNotFoundError),
}

#[derive(Debug)]
pub enum Lookup<T> {
    Found(T),
    NotFound,
}

pub struct UserService {
    pub users: Arc<dyn UserRepository>,
}

impl UserService {
    pub async fn get_one_by_id(&self, id: UserId) -> Result<User, UserServiceError> {
        self.users
            .find_one_by_id(id)
            .await?
            .ok_or_else(|| UserNotFoundError { user_id: id }.into())
    }

    pub async fn try_get_one_by_id(
        &self,
        id: UserId,
    ) -> Result<Lookup<User>, RepositoryError> {
        Ok(match self.users.find_one_by_id(id).await? {
            Some(user) => Lookup::Found(user),
            None => Lookup::NotFound,
        })
    }
}
```

## 3. Notes

- `Result<Option<User>, RepositoryError>` is the `find` contract; `Option` is absence, `Err` is failure.
- `find_many` returns an empty `Vec`, never `None` for "no matches".
- `Lookup` is the explicit `tryGet` contract, distinct from `Option`.
- `update_one_by_id` returns a boolean; bulk updates return a count.
- A bare `update_one(user)` is permitted only when `user.id` is required and validated; `update_one_by_id` is preferred.
- If the project uses an All-based profile, map to `find_by_id`, `find_all`, `find_all_by_status`, `update_by_id`, and `remove_by_id`; keep `insert_one`/`insert_many`.
- Put `get` and `try_get` in the service; the repository exposes `find`.
