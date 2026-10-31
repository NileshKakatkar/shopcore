# ShopCore

ShopCore is an e-commerce backend built as a set of independently versioned Go services. Every service has its own Go module, Dockerfile, and Git repository. The workspace folder itself has no Git repository, root Go module, or `go.work` file.

## Services

- `gateway-service`: Fiber API gateway, Redis rate limiting, and reverse proxy
- `user-service`: registration, login, JWT issuing, user profile, and gRPC user-details server
- `product-service`: product and category management, inventory reservation, and gRPC product-details server
- `order-service`: order management, gRPC calls to User and Product, inventory reservation, and Kafka outbox publisher
- `email-service`: Kafka consumer and email delivery

## Shared modules

- `shared/grpc`: Protocol Buffers definitions and generated gRPC code used by User, Product, and Order
- `shared/events`: Kafka event contracts used by Order and Email

The shared modules are siblings of the service repositories. User, Product, and Order use `shared/grpc`; Order and Email use `shared/events`. Keep the workspace layout intact when building the complete stack locally.

## Repository layout

```text
ShopCore/
├── gateway-service/
├── user-service/
├── product-service/
├── order-service/
├── email-service/
├── shared/
│   ├── grpc/
│   └── events/
└── deployment/
```

Each service, shared module, and deployment folder has its own `.git` directory. There is no root `.git` directory.

## Run the stack

```sh
cd deployment
cp .env.example .env
sh scripts/generate-jwt-keys.sh
docker compose up --build
```

The API gateway is available at `http://localhost:8080`. User and Product expose internal gRPC endpoints on ports `50051` and `50052`. Order writes `order.placed` events to Kafka through a transactional outbox, and Email consumes those events.

## Run a service locally

Keep the shared modules beside the service directories. Start PostgreSQL, Redis, and Kafka first, then run a service from its own directory:

```sh
cd user-service
go mod download
go run ./cmd/user
```

Set the environment variables shown in `deployment/compose.yaml` before starting each service.

## API routes

- `POST /api/v1/auth/register`
- `POST /api/v1/auth/login`
- `GET /api/v1/users/me`
- `GET /api/v1/categories`
- `GET /api/v1/products`
- `GET /api/v1/products/:id`
- `POST /api/v1/orders`
- `GET /api/v1/orders`
- `GET /api/v1/orders/:id`

All public API requests go through the gateway on port `8080`.
