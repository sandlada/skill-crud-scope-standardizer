# TypeScript Template

Status: Supporting template
Normative source: [../../core/00-rules.md](../../core/00-rules.md)

## 1. Repository contract

```ts
export type UserId = string & { readonly __brand: "UserId" };

export type UserStatus = "active" | "suspended" | "removed";

export interface User {
  readonly id: UserId;
  readonly email: string;
  readonly status: UserStatus;
  readonly createdAt: Date;
}

export interface UserPatch {
  readonly email?: string;
  readonly status?: UserStatus;
}

export interface FindManyOptions {
  readonly limit?: number;
  readonly offset?: number;
  readonly orderBy?: readonly {
    readonly field: "createdAt" | "id";
    readonly direction: "asc" | "desc";
  }[];
}

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

## 2. Service boundary

```ts
export class UserNotFoundError extends Error {
  constructor(readonly userId: UserId) {
    super(`User not found: ${userId}`);
    this.name = "UserNotFoundError";
  }
}

export type Result<T, E> =
  | { readonly ok: true; readonly value: T }
  | { readonly ok: false; readonly error: E };

export class UserService {
  constructor(private readonly users: UserRepository) {}

  async getOneById(id: UserId): Promise<User> {
    const user = await this.users.findOneById(id);
    if (user === null) throw new UserNotFoundError(id);
    return user;
  }

  async tryGetOneById(
    id: UserId,
  ): Promise<Result<User, UserNotFoundError>> {
    const user = await this.users.findOneById(id);
    return user === null
      ? { ok: false, error: new UserNotFoundError(id) }
      : { ok: true, value: user };
  }
}
```

## 3. Notes

- Use `null` consistently for `findOne` absence; do not mix `null` and `undefined` in one contract.
- `findMany` returns `[]`, never `null`.
- `tryGet` returns a discriminated result; a bare nullable value is the `find` contract.
- `updateOneById` returns whether a record was updated; use a count for bulk updates.
- Validate the branded `UserId` at the boundary before calling the repository.
- If the project adopts an All-based profile, map to `findById`, `findAll`, `findAllByStatus`, `updateById`, and `removeById`, retaining `insertOne`/`insertMany`.
- Put `get` and `tryGet` in the service; the repository exposes `find`.
