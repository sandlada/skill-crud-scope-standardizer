# Go Template

Status: Supporting template
Normative source: [../../core/00-rules.md](../../core/00-rules.md)

## 1. Repository contract

```go
package users

import (
	"context"
	"time"
)

type UserID string

type UserStatus string

const (
	UserStatusActive    UserStatus = "active"
	UserStatusSuspended UserStatus = "suspended"
	UserStatusRemoved   UserStatus = "removed"
)

type User struct {
	ID        UserID
	Email     string
	Status    UserStatus
	CreatedAt time.Time
}

type UserPatch struct {
	Email  *string
	Status *UserStatus
}

type FindManyOptions struct {
	Limit   *int
	Offset  int
	OrderBy string
}

type UserRepository interface {
	InsertOne(ctx context.Context, user User) (User, error)
	InsertMany(ctx context.Context, users []User) ([]User, error)
	FindOneByID(ctx context.Context, id UserID) (*User, error)
	FindMany(ctx context.Context, options FindManyOptions) ([]User, error)
	FindManyByStatus(ctx context.Context, status UserStatus, options FindManyOptions) ([]User, error)
	UpdateOneByID(ctx context.Context, id UserID, patch UserPatch) (bool, error)
	UpdateManyByStatus(ctx context.Context, status UserStatus, patch UserPatch) (int, error)
	RemoveOneByID(ctx context.Context, id UserID) (bool, error)
	RemoveManyByStatus(ctx context.Context, status UserStatus) (int, error)
}
```

## 2. Service boundary

```go
package users

import (
	"context"
	"errors"
)

var ErrUserNotFound = errors.New("user not found")

type UserNotFoundError struct {
	UserID UserID
}

func (e *UserNotFoundError) Error() string {
	return "user not found: " + string(e.UserID)
}

func (e *UserNotFoundError) Unwrap() error { return ErrUserNotFound }

type LookupResult[T any] struct {
	Found bool
	Value T
}

type UserService struct {
	users UserRepository
}

func NewUserService(users UserRepository) *UserService {
	return &UserService{users: users}
}

func (s *UserService) GetOneByID(ctx context.Context, id UserID) (User, error) {
	user, err := s.users.FindOneByID(ctx, id)
	if err != nil {
		return User{}, err
	}
	if user == nil {
		return User{}, &UserNotFoundError{UserID: id}
	}
	return *user, nil
}

func (s *UserService) TryGetOneByID(ctx context.Context, id UserID) (LookupResult[User], error) {
	user, err := s.users.FindOneByID(ctx, id)
	if err != nil {
		return LookupResult[User]{}, err
	}
	if user == nil {
		return LookupResult[User]{Found: false}, nil
	}
	return LookupResult[User]{Found: true, Value: *user}, nil
}
```

## 3. Notes

- `(*User, error)` represents `find`: `nil` is absence, `err` is failure.
- `findMany` returns a non-nil empty slice, never `nil` for "no matches".
- `tryGet` returns `LookupResult`, distinct from `find`.
- `UpdateOneByID` returns a boolean; bulk updates return a count.
- A bare `UpdateOne(ctx, user)` is permitted only when `user.ID` is required and validated; `UpdateOneByID` is preferred.
- If the project uses an All-based profile, map to `FindByID`, `FindAll`, `FindAllByStatus`, `UpdateByID`, and `RemoveByID`; keep `InsertOne`/`InsertMany`.
- Put `Get` and `TryGet` in the service; the repository exposes `Find`.
