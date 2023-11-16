# Awesome Docker Services

Welcome to Awesome Docker Services! This repository houses Dockerfiles and docker-compose files for various developer services, facilitating easy setup for development and production environments.

- [Golang](#golang)
- [Redis](#redis)
- [RabbitMQ](#rabbitmq)
- [PostgreSQL](#postgresql)

## Usage Instructions

Use this repository for reference when setting up your own Dockerfiles and docker-compose files.

## Golang

#### Dockerfile.dev

Use this Dockerfile to set up a Golang development environment with hot reload.

```dockerfile
FROM golang:1.20-alpine

WORKDIR /app

ENV CGO_ENABLED=0
ENV GOOS=linux
ENV GOARCH=amd64

# install hot reload tool
RUN go install github.com/githubnemo/CompileDaemon@v1.4.0

COPY . .

# change `./cmd/app` to your main package
ENTRYPOINT CompileDaemon -log-prefix=false -build="go build -mod vendor ./cmd/app" -command="./app"
```

#### Dockerfile.prod

Dockerfile optimized for Golang production environment.

```dockerfile
# stage 1: build binary
FROM golang:1.20-alpine as build

WORKDIR /app

COPY . .

RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 \
    go build -mod vendor -o /bin/app ./cmd/app

# stage 2: run binary
FROM alpine:3.12
RUN apk --update add ca-certificates tzdata

WORKDIR /app

COPY --from=build /bin/app ./app

ENTRYPOINT [ "./app" ]
```

#### docker-compose.yml

```yaml
version: '3.9'
services:
  app: # change your service name
    build:
      context: . # path to Dockerfile
      dockerfile: ./Dockerfile.dev # change to `Dockerfile.prod` for production
    ports:
      - "8080:8080" # host:container
    volumes:
      - .:/app # mount current directory to /app in container
```

## Redis

#### docker-compose.yml

```yaml
version: '3.9'
services:
  redis:
    image: "redis:7-alpine"
    ports:
      - "6379:6379" # host:container
```

## RabbitMQ

#### docker-compose.yml

```yaml
version: '3.9'
services:
  rabbitmq:
    image: "rabbitmq:3-management-alpine"
    environment:
      RABBITMQ_DEFAULT_USER: "guest"
      RABBITMQ_DEFAULT_PASS: "guest"
      RABBITMQ_DEFAULT_VHOST: "/"
    ports:
      - 15672:15672 # host:container management
      - 5672:5672 # host:container service
```

## PosgreSQL

#### docker-compose.yml

```yaml
version: '3.9'
services:
  postgres:
    image: "postgres:16.0-alpine3.18"
    ports:
      - "5432:5432" # host:container
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: postgres
```

## Contributing

Contributions to add more services, improve existing configurations, or provide enhancements are welcome! Please follow the guidelines in [CONTRIBUTING.md](./CONTRIBUTING.md).

## License

This repository is licensed under the [MIT License](./LICENSE).

## Support and Feedback

For any questions, issues, or feedback, please create an issue.
