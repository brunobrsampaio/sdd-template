# Containers — DevOps

> Guia de boas práticas para Dockerfiles, Docker Compose e segurança de imagens.
> Complementa o [`spec.md`](./spec.md) deste mesmo diretório.

---

## Regras Gerais [PADRÃO]

As regras de imagem (multi-stage build, usuário não-root em produção, base fixa `alpine`/`slim` sem `latest` e `.dockerignore` obrigatório) estão em "Docker" no [`spec.md`](./spec.md). Este guia adiciona:

- **Camadas:** ordenadas por frequência de mudança — dependências antes de código-fonte
- **Secrets:** proibido passar secrets via `ARG` ou `ENV` no build — use montagem de secrets ou variáveis de runtime
- **Tamanho:** imagem final contém apenas o necessário para execução — sem ferramentas de build, devDependencies ou docs

> Para regras de escaneamento de vulnerabilidades e auditoria de imagens, veja "Secrets e Segurança" no [`spec.md`](./spec.md).

---

## Exemplo: Dockerfile multi-stage (Node.js)

```dockerfile
# Dockerfile

FROM node:20-alpine AS builder

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci

COPY . .
RUN npm run build

# ---

FROM node:20-alpine AS runner

WORKDIR /app

RUN addgroup -S appgroup && adduser -S appuser -G appgroup

COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./

USER appuser

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/healthz || exit 1

CMD ["node", "dist/server.js"]
```

---

## Exemplo: Dockerfile multi-stage (PHP/Laravel)

```dockerfile
# Dockerfile

FROM composer:2 AS vendor

WORKDIR /app

COPY composer.json composer.lock ./
RUN composer install --no-dev --no-scripts --no-interaction --prefer-dist

COPY . .
RUN composer dump-autoload --optimize

# ---

FROM php:8.3-fpm-alpine AS runner

WORKDIR /var/www/html

RUN addgroup -S appgroup && adduser -S appuser -G appgroup

RUN apk add --no-cache libpq-dev \
    && docker-php-ext-install pdo_pgsql opcache

COPY --from=vendor /app /var/www/html

RUN chown -R appuser:appgroup /var/www/html/storage /var/www/html/bootstrap/cache

USER appuser

EXPOSE 9000

HEALTHCHECK --interval=30s --timeout=5s --start-period=15s --retries=3 \
  CMD php artisan health:check || exit 1

CMD ["php-fpm"]
```

---

## Exemplo: docker-compose.yml para desenvolvimento local

```yaml
# docker-compose.yml

services:
  app:
    build:
      context: .
      target: builder
    ports:
      - "3000:3000"
    volumes:
      - .:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://dev:dev@postgres:5432/app_dev
      - REDIS_URL=redis://redis:6379
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy

  postgres:
    image: postgres:16-alpine
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: dev
      POSTGRES_DB: app_dev
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U dev"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redisdata:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  pgdata:
  redisdata:
```

> Credenciais no Compose são apenas para desenvolvimento local. Em staging/production, variáveis vêm do cofre da plataforma — veja [`environments.md`](./environments.md).

---

## Exemplo: .dockerignore

```dockerignore
# Controle de versão
.git
.gitignore

# Dependências locais
node_modules
vendor

# Variáveis de ambiente
.env
.env.*
!.env.example

# Testes e documentação
tests/
__tests__/
*.test.ts
*.spec.ts
coverage/
docs/

# Artefatos de build local
dist/
build/

# IDEs e editores
.vscode/
.idea/
*.swp
*.swo

# Docker
Dockerfile
docker-compose*.yml
.dockerignore

# CI
.github/
.gitlab-ci.yml
bitbucket-pipelines.yml

# OS
.DS_Store
Thumbs.db
```

---

## Exemplo: Health checks no container

Health checks verificam se o container está operando corretamente. Dois conceitos distintos:

- **Liveness:** "o processo está vivo?" — reinicia o container se falhar
- **Readiness:** "o processo está pronto para receber tráfego?" — remove do balanceador se falhar

```dockerfile
# Liveness via HEALTHCHECK no Dockerfile
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/healthz || exit 1
```

```yaml
# docker-compose.yml — healthcheck no serviço
services:
  app:
    build: .
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:3000/healthz"]
      interval: 30s
      timeout: 5s
      start_period: 10s
      retries: 3
```

> Para implementação dos endpoints `/healthz` e `/readyz` na aplicação, veja [`monitoring.md`](./monitoring.md).

---

## Anti-patterns (o que evitar)

```dockerfile
# ❌ Imagem base sem versão fixa
FROM node:latest

# ✅ Versão fixa com variante leve
FROM node:20-alpine
```

```dockerfile
# ❌ Rodar como root
FROM node:20-alpine
WORKDIR /app
COPY . .
CMD ["node", "server.js"]

# ✅ Usuário não-privilegiado
FROM node:20-alpine
WORKDIR /app
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
COPY . .
USER appuser
CMD ["node", "server.js"]
```

```dockerfile
# ❌ Copiar node_modules do host para dentro da imagem
COPY . .
# node_modules local pode conter binários incompatíveis com o SO do container

# ✅ Instalar dentro do container
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
```

```dockerfile
# ❌ npm install em vez de npm ci
RUN npm install
# Pode gerar lockfile diferente e instalar versões inesperadas

# ✅ npm ci respeita o lockfile exatamente
RUN npm ci
```

```dockerfile
# ❌ Secrets como ARG no build
ARG DATABASE_URL
ARG API_SECRET
ENV DATABASE_URL=$DATABASE_URL
ENV API_SECRET=$API_SECRET

# ✅ Secrets via variável de ambiente no runtime (docker run ou compose)
# Dockerfile não contém nenhum secret — valores vêm de fora
ENV NODE_ENV=production
```

```dockerfile
# ❌ Camadas em ordem errada — cache invalidado a cada mudança de código
COPY . .
RUN npm ci

# ✅ Dependências primeiro — cache aproveitado quando só o código muda
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
```
