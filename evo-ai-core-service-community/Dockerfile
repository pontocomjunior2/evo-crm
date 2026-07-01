# Build stage
FROM golang:1.24.4-alpine AS builder

WORKDIR /app

# Cache bust 2026-07-01-v3
ARG CACHEBUST=3

# Instala dependências do sistema
RUN apk add --no-cache git

# Instala golang-migrate
RUN go install -tags 'postgres' github.com/golang-migrate/migrate/v4/cmd/migrate@latest

# Copia os arquivos de dependência
COPY go.mod go.sum ./

# Baixa as dependências
RUN go mod download

# Copia o código fonte
COPY . .

# Compila a aplicação
RUN CGO_ENABLED=0 GOOS=linux go build -o /app/main ./cmd/api

# Final stage
FROM alpine:latest

WORKDIR /app

# Instala ca-certificates
RUN apk --no-cache add ca-certificates

# Copia o binário compilado
COPY --from=builder /app/main ./main
COPY --from=builder /go/bin/migrate ./migrate

# Copia migrations como wildcard (mais robusto)
COPY ./migrations/ ./migrations/
COPY entrypoint.sh ./entrypoint.sh

# Permissões
RUN chmod +x ./main ./migrate ./entrypoint.sh

# Expõe a porta
EXPOSE 5555

# Executa
CMD ["./entrypoint.sh"]
