# Proto Registry

Central repository for all `.proto` files defining the gRPC contracts between Volontariapp microservices.

## 📂 Structure

```
proto/
├── common/              # Shared messages used across services
│   ├── pagination.proto
│   └── timestamp.proto
├── user/                # ms-user service definitions
│   └── user.proto
├── event/               # ms-event service definitions
│   └── event.proto
└── post/                # ms-post service definitions
    └── post.proto
```

## 🎯 Conventions

### Package Naming

All proto packages follow the pattern: `volontariapp.<domain>.v1`

```protobuf
package volontariapp.user.v1;
```

### File Organization

- **One service per file** — each `.proto` file defines a single gRPC service.
- **`common/`** — shared message types (pagination, timestamps, etc.) imported by other protos.
- **Versioning** — all services are namespaced under `v1`. When breaking changes are introduced, create a `v2` directory alongside `v1`.

### Naming Conventions (Proto Style Guide)

| Element | Convention                              | Example                             |
| ------- | --------------------------------------- | ----------------------------------- |
| Package | `lowercase.dot.separated`               | `volontariapp.user.v1`              |
| Service | `PascalCase` + `Service` suffix         | `UserService`                       |
| RPC     | `PascalCase` verb                       | `GetUser`, `CreateUser`             |
| Message | `PascalCase`                            | `UserResponse`, `CreateUserRequest` |
| Field   | `snake_case`                            | `first_name`, `created_at`          |
| Enum    | `PascalCase` name, `UPPER_SNAKE` values | `UserRole`, `USER_ROLE_ADMIN`       |

## 🔗 Usage

This repository is consumed as a **Git submodule** by each service:

```bash
git submodule add git@github.com:Volontariapp/proto-registry.git proto-registry
```

Services reference the `.proto` files using the submodule path:

```typescript
// NestJS gRPC client example
{
  transport: Transport.GRPC,
  options: {
    package: 'volontariapp.user.v1',
    protoPath: join(__dirname, '../../proto-registry/proto/user/user.proto'),
  },
}
```

## 🛠️ Development

### Linting

```bash
buf lint
```

### Breaking Change Detection

```bash
buf breaking --against '.git#branch=main'
```
