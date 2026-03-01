# Proto Registry

Central repository for all `.proto` files defining the gRPC contracts between Volontariapp microservices.

## 📂 Structure

```
proto/volontariapp/
├── common/
│   ├── pagination.proto        # PaginationRequest / PaginationResponse
│   └── timestamp.proto         # DateRange (wraps google.protobuf.Timestamp)
├── event/
│   ├── event.proto             # Event entity
│   ├── event.requests.proto    # Get / List / Create / Update / Delete requests
│   ├── event.responses.proto   # Get / List / Create / Update / Delete responses
│   └── event.services.proto    # EventService RPC definitions
├── post/
│   ├── post.proto              # Post entity
│   ├── post.requests.proto
│   ├── post.responses.proto
│   └── post.services.proto     # PostService RPC definitions
└── user/
    ├── user.proto              # User entity
    ├── user.requests.proto
    ├── user.responses.proto
    └── user.services.proto     # UserService RPC definitions
```

## 🎯 Conventions

### Package Naming

All proto packages follow the pattern: `volontariapp.<domain>`

```protobuf
package volontariapp.user;
```

### File Organization

Each domain has **4 files** following dot notation:

| File                   | Contains                                 |
| ---------------------- | ---------------------------------------- |
| `<domain>.proto`       | Entity message (`User`, `Event`, `Post`) |
| `<domain>.requests.proto`  | Request messages (`GetUserRequest`, etc.)  |
| `<domain>.responses.proto` | Response messages (`GetUserResponse`, etc.) |
| `<domain>.services.proto`  | gRPC `service` definition with RPCs       |

### Naming Conventions

| Element | Convention                              | Example                             |
| ------- | --------------------------------------- | ----------------------------------- |
| Package | `lowercase.dot.separated`               | `volontariapp.user`                 |
| Service | `PascalCase` + `Service` suffix         | `UserService`                       |
| RPC     | `PascalCase` verb                       | `GetUser`, `CreateUser`             |
| Message | `PascalCase`                            | `UserResponse`, `CreateUserRequest` |
| Field   | `snake_case`                            | `first_name`, `created_at`          |
| Enum    | `PascalCase` name, `UPPER_SNAKE` values | `UserRole`, `USER_ROLE_ADMIN`       |

## ⚙️ Code Generation

### buf.gen.yaml

TypeScript types are generated using [ts-proto](https://github.com/stephenh/ts-proto) with the following options:

| Option                    | Purpose                                             |
| ------------------------- | --------------------------------------------------- |
| `nestJs=true`             | Generates NestJS-compatible service interfaces      |
| `useDate=true`            | Maps `google.protobuf.Timestamp` to native `Date`   |
| `importSuffix=.js`        | ESM-compatible imports for `nodenext` resolution     |
| `exportCommonSymbols=false`| Prevents duplicate `protobufPackage` export conflicts |
| `outputClientImpl=false`  | No gRPC client stubs (services handle this)          |

### Generated output

Running `buf generate` produces TypeScript files in `gen/ts/` with:
- Entity interfaces (`Event`, `Post`, `User`)
- Request/Response interfaces
- NestJS service interfaces (`EventServiceClient`, `EventServiceController`)
- NestJS decorator functions (`EventServiceControllerMethods()`)
- Service name constants (`EVENT_SERVICE_NAME`)

## 🔄 CI/CD Pipeline

### On Pull Request
1. **check-proto-changes** — detects if any `.proto` files were modified
2. **buf lint** — runs only if proto files changed

### On Push to Main
1. **check-proto-changes** — detects if any `.proto` files were modified
2. **buf lint** — runs only if proto files changed
3. **sync** — generates TypeScript types and creates a PR on `npm-packages` with updated `@volontariapp/contracts`

### Manually
Use `workflow_dispatch` to trigger the full pipeline.

## 🛠️ Development

### Linting

```bash
buf lint
```

### Generate TypeScript locally

```bash
buf generate
```

### Breaking Change Detection

```bash
buf breaking --against '.git#branch=main'
```

## 🔗 Usage

This repository is consumed as a **Git submodule** by each service and by `npm-packages`.

The generated TypeScript types are published as `@volontariapp/contracts` via the automated sync pipeline.

```typescript
import { User, CreateUserRequest, UserServiceController } from '@volontariapp/contracts';
```
