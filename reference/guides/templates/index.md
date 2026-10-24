# Template Index

Status: Supporting index
Normative source: [../../core/00-rules.md](../../core/00-rules.md)

Load one template for the target language. Do not load the whole directory.

| Language | Template | Member casing |
| --- | --- | --- |
| TypeScript | [typescript.md](typescript.md) | camelCase |
| C# | [csharp.md](csharp.md) | PascalCase; `Async` suffix |
| Java | [java.md](java.md) | camelCase |
| Kotlin | [kotlin.md](kotlin.md) | camelCase; `suspend` |
| Python | [python.md](python.md) | snake_case |
| Go | [go.md](go.md) | PascalCase; `ID` acronym |
| Rust | [rust.md](rust.md) | snake_case |

All templates use the default profile. An All-based profile maps single reads to `findById`, plural reads to `findAll`/`findAllBy...`, single updates to `updateById` or identity-bearing `update(user)`, and removals to `removeById`/`removeAllByStatus`; `insertOne` and `insertMany` are retained.
