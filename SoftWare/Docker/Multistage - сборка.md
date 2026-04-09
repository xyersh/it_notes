
## Пример production ready docker-файла для go-прилоджения
```dockerfile
# Stage 1: сборка зависимостей. 
FROM golang:1.25.7-alpine AS builder 
WORKDIR /build 
COPY go.mod go.sum ./ 
RUN go mod download 
COPY . . 
RUN CGO_ENABLED=0 GOOS=linux go build -o myapp . 

# Stage 2: финальный образ. 
FROM alpine:3.21 

ARG BUILD_TIME 
ARG VERSION 

LABEL com.express42.compiler="Golang" \ 
org.opencontainers.image.version=${VERSION} \ org.opencontainers.image.authors="andrey@express42.com" \ org.opencontainers.image.description="Backend API" \ org.opencontainers.image.created=${BUILD_TIME} 

WORKDIR /opt/app 
RUN apk add --no-cache postgresql17-client && \ 
addgroup -S appuser --gid 2100 && \ 
adduser -S appuser --uid 2101 -G appuser 
COPY --chown=appuser:appuser --from=builder /build/myapp . 

USER appuser:appuser 

# ENTRYPOINT запускает приложение. 
ENTRYPOINT ["./myapp"]
```

## Пояснение к содержимому файла

### Этап 1: Сборка приложения (`builder`)

```dockerfile
FROM golang:1.25.7-alpine AS builder
```
- **`FROM`** — указывает базовый образ. Здесь используется официальный образ Go на базе Alpine Linux.
- **`AS builder`** — даёт имя этому этапу сборки, чтобы позже можно было скопировать из него артефакты.