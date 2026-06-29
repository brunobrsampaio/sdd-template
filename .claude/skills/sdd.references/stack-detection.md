# Detecção de Stack (Brownfield)

> Arquivo de referência usado pela skill `sdd.adopt`.
> Fonte canônica de **como detectar a stack real de um projeto já existente** a partir dos
> seus arquivos — manifestos, configs, diretórios e CI — e como inferir cada campo das specs.
>
> **Princípio central — stack-agnóstico:** as tabelas abaixo cobrem os ecossistemas mais comuns,
> mas a lista é **aberta**. Sempre que um sinal não estiver nas tabelas, aplique o
> **Fallback (stack desconhecida)**: leia o manifesto, liste as dependências e infira o valor
> real como **texto livre** — nunca force um valor para uma opção do catálogo do
> `question-flows.md`.

---

## Modelo de evidência e confiança

Para **cada campo** detectado, registre uma tripla:

```
<valor> · <confiança> · <evidência>
```

- **valor:** o nome real da tecnologia (rótulo do `question-flows.md` se coincidir; senão, texto livre).
- **confiança:**
  - **alta** — sinal direto e inequívoco (dependência declarada no manifesto, arquivo de config
    dedicado, diretório canônico). Ex: `react` em `package.json`, `drizzle.config.ts`.
  - **média** — sinal indireto ou que admite mais de uma leitura. Ex: presença de `.env` com
    `DATABASE_URL=postgres://...` sugere PostgreSQL, mas não confirma o ORM.
  - **baixa** — heurística fraca ou ausência de sinal forte (palpite por convenção).
- **evidência:** o **arquivo** (e linha, quando útil) que originou o valor. Ex: `package.json:24`.

**Regra de confirmação:** valores **alta** podem ser apresentados como detectados (o usuário só
confirma). Valores **média/baixa** e campos **não detectados** entram no preenchimento de lacunas
(`AskUserQuestion` ou texto livre) — ver Fases 4 e 5 do `sdd.adopt`.

---

## Detecção de camadas presentes

Uma camada é considerada **presente** se houver ao menos um sinal forte dela:

| Camada | Sinais de presença |
|--------|--------------------|
| **Frontend** | Dep de framework de UI (`react`, `vue`, `@angular/core`, `svelte`, `solid-js`); `index.html` + bundler (`vite`, `webpack`); diretórios `src/components`, `pages/`, `app/`; config de estilo (`tailwind.config.*`). |
| **Backend** | Manifesto de servidor + dep de framework web (ver tabela Backend); diretórios `src/controllers`, `app/Http`, `routes/`, `cmd/`, `api/`; ponto de entrada de servidor (`main.go`, `server.ts`, `app.py`, `index.php`). |
| **Database** | Driver/ORM nas deps; diretório de migrations; serviço de banco em `docker-compose.yml`; `DATABASE_URL`/`DB_*` em `.env*`; `schema.prisma`, `*.sql`. |
| **DevOps** | `.github/workflows/`, `.gitlab-ci.yml`, `.circleci/`, `Jenkinsfile`, `azure-pipelines.yml`; `Dockerfile`, `docker-compose.yml`, `k8s/`, `*.tf`; configs de hosting (`fly.toml`, `vercel.json`, etc.). |

> Uma camada **sem nenhum sinal** não é ativada. Não invente camadas — registre apenas o que tem evidência.

---

## Manifesto raiz → ecossistema / linguagem / runtime

| Manifesto / arquivo | Ecossistema | Como ler runtime/versão |
|---------------------|-------------|--------------------------|
| `package.json` | Node.js (JS/TS) | `engines.node`; `.nvmrc`, `.node-version`, `.tool-versions`; `tsconfig.json` ⇒ TypeScript |
| `deno.json` / `deno.jsonc` | Deno (TS) | versão no CI ou `deno.lock` |
| `composer.json` | PHP | `require.php`; `.tool-versions` |
| `requirements.txt` / `pyproject.toml` / `Pipfile` | Python | `python_requires` / `[tool.poetry.dependencies].python`; `.python-version` |
| `go.mod` | Go | linha `go 1.xx` no próprio `go.mod` |
| `pom.xml` / `build.gradle(.kts)` | Java / Kotlin | `maven.compiler.*` / `sourceCompatibility`; `.sdkmanrc` |
| `Gemfile` | Ruby | `ruby "x.y"` no Gemfile; `.ruby-version` |
| `Cargo.toml` | Rust | `rust-version`; `rust-toolchain.toml` |
| `*.csproj` / `*.sln` / `global.json` | .NET (C#) | `<TargetFramework>`; `global.json` |
| `pubspec.yaml` | Dart / Flutter | `environment.sdk` |
| `mix.exs` | Elixir | `elixir` em `mix.exs`; `.tool-versions` |

> **Monorepo:** se houver múltiplos manifestos (ex: `apps/web/package.json` + `apps/api/go.mod`),
> trate cada um como uma camada distinta e registre o caminho na evidência.

---

## Backend — dependências → framework / ORM / validação / auth

> Leia a seção de dependências do manifesto. As tabelas listam os sinais mais comuns; para
> qualquer dep fora delas, use o **Fallback**.

### Frameworks de servidor (por ecossistema)

| Ecossistema | Dep / sinal → Framework |
|-------------|--------------------------|
| Node.js | `fastify`→Fastify · `@nestjs/core`→NestJS · `express`→Express · `hono`→Hono · `koa`→Koa |
| PHP | `laravel/framework`→Laravel · `symfony/framework-bundle`→Symfony · `slim/slim`→Slim |
| Python | `django`→Django · `fastapi`→FastAPI · `flask`→Flask |
| Go | `gin-gonic/gin`→Gin · `labstack/echo`→Echo · `gofiber/fiber`→Fiber · `go-chi/chi`→chi |
| Java/Kotlin | `spring-boot-starter`→Spring Boot · `quarkus`→Quarkus · `ktor`→Ktor |
| Ruby | `rails`→Rails · `sinatra`→Sinatra |
| Rust | `actix-web`→Actix · `axum`→Axum · `rocket`→Rocket |
| .NET | `Microsoft.AspNetCore.*`→ASP.NET Core |

### ORM / acesso a dados

| Ecossistema | Sinal → ORM |
|-------------|--------------|
| Node.js | `schema.prisma`/`prisma`→Prisma · `drizzle.config.*`/`drizzle-orm`→Drizzle · `typeorm`→TypeORM · `sequelize`→Sequelize · `@mikro-orm/*`→MikroORM · `knex`→Knex |
| PHP | Laravel⇒Eloquent · `doctrine/orm`→Doctrine · `cycle/orm`→Cycle |
| Python | `sqlalchemy`→SQLAlchemy · Django⇒Django ORM · `tortoise-orm`→Tortoise |
| Go | `gorm.io/gorm`→GORM · `ent`→Ent · `sqlc`→sqlc · `sqlx`→sqlx |
| Java/Kotlin | `spring-boot-starter-data-jpa`/`hibernate`→Hibernate (JPA) · `jooq`→jOOQ |
| Ruby | Rails⇒ActiveRecord |
| Rust | `diesel`→Diesel · `sea-orm`→SeaORM · `sqlx`→SQLx |
| .NET | `Microsoft.EntityFrameworkCore`→Entity Framework Core |

### Validação

`zod`→Zod · `joi`→Joi · `class-validator`→class-validator · `yup`→Yup · `valibot`→Valibot ·
`pydantic`→Pydantic · `go-playground/validator`→validator · `hibernate-validator`→Hibernate Validator ·
Laravel⇒Laravel Validator · `symfony/validator`→Symfony Validator. Fora disso → **Fallback**.

### Autenticação

`jsonwebtoken`/`@fastify/jwt`→JWT · `passport`→Passport · `lucia`→Lucia · `next-auth`→NextAuth ·
`laravel/sanctum`→Sanctum · `laravel/passport`→Passport (PHP) · `djangorestframework-simplejwt`→JWT (DRF) ·
`devise`→Devise · `spring-boot-starter-security`→Spring Security. Fora disso (ou middleware caseiro) → **Fallback**.

---

## Frontend — dependências → campos

| Campo | Sinais |
|-------|--------|
| **Framework** | `react`+`react-dom`→React · `next`→Next.js · `vue`→Vue · `@angular/core`→Angular · `svelte`/`@sveltejs/kit`→Svelte/SvelteKit · `solid-js`→Solid · `nuxt`→Nuxt |
| **Estilização** | `tailwind.config.*`→Tailwind · `styled-components`→Styled Components · `*.module.css`→CSS Modules · `sass`→Sass/SCSS · `@emotion/*`→Emotion · `@vanilla-extract/*`→vanilla-extract |
| **Estado global** | `zustand`→Zustand · `@reduxjs/toolkit`→Redux Toolkit · `jotai`→Jotai · `recoil`→Recoil · `pinia`/`vuex`→Pinia/Vuex · `@ngrx/store`→NgRx |
| **Estado de servidor** | `@tanstack/react-query`→TanStack Query · `swr`→SWR · `@apollo/client`→Apollo · `urql`→urql |
| **Formulários** | `react-hook-form`→React Hook Form · `formik`→Formik · `@tanstack/react-form`→TanStack Form · `vee-validate`→VeeValidate |
| **Roteamento** | `react-router-dom`→React Router · `@tanstack/react-router`→TanStack Router · `wouter`→Wouter · Next: `app/`→App Router, `pages/`→Pages Router · `vue-router`→Vue Router |
| **UI Library** | `components.json`→shadcn/ui · `@headlessui/*`→Headless UI · `daisyui`→DaisyUI · `@mui/*`→MUI · `antd`→Ant Design · `@chakra-ui/*`→Chakra · `@mantine/*`→Mantine |

> Se o framework não for React/Next nem usar Tailwind, o passo de **UI Library** não se aplica
> (registre "Não se aplica" salvo evidência de uma lib específica).

---

## Database — campos

| Campo | Sinais |
|-------|--------|
| **Banco principal** | Driver: `pg`/`postgres`→PostgreSQL · `mysql2`/`mysql`→MySQL · `mongodb`/`mongoose`→MongoDB · `sqlite3`/`better-sqlite3`→SQLite · `redis`→Redis (cache). Serviço em `docker-compose.yml` (image `postgres`, `mysql`, `mongo`). Driver Python (`psycopg`, `pymysql`), Go (`lib/pq`, `go-sql-driver/mysql`), etc. |
| **ORM / Query builder** | Herdar do Backend (ver tabela ORM acima). Se houver Backend ativo, **não re-detectar** — usar o ORM do Backend. |
| **Migrations** | `prisma/migrations/`→Prisma Migrate · `drizzle/`+`drizzle.config`→Drizzle Kit · `database/migrations/`→Laravel/Doctrine · `migrations/` (Alembic `alembic.ini`)→Alembic · `db/migrate/`→Rails · `migrations/*.sql`→ferramenta SQL (Flyway/Liquibase se `flyway.conf`/`liquibase`) |
| **Banco de desenvolvimento** | `docker-compose.yml` com serviço de banco→Docker local · `*.sqlite`/`*.db` versionado→SQLite local · `DATABASE_URL` remoto em `.env.example`→remoto compartilhado |
| **Banco de teste** | Serviço de banco em workflow de CI→Docker Compose isolado · SQLite em config de teste→SQLite em memória · ausência→"Não se aplica" |

---

## DevOps — campos

| Campo | Sinais |
|-------|--------|
| **CI/CD** | `.github/workflows/*`→GitHub Actions · `.gitlab-ci.yml`→GitLab CI · `.circleci/config.yml`→CircleCI · `Jenkinsfile`→Jenkins · `azure-pipelines.yml`→Azure Pipelines |
| **Containerização** | `Dockerfile`+`docker-compose.yml`→Docker + Compose · `Containerfile`→Podman · `k8s/`,`*.yaml` com `kind:`→Kubernetes · `helm/`→Helm |
| **Hospedagem** | `fly.toml`→Fly.io · `railway.json`/`railway.toml`→Railway · `vercel.json`→Vercel · `netlify.toml`→Netlify · `app.yaml`→GCP App Engine · `*.tf` com provider aws/gcp/azure→cloud correspondente · `serverless.yml`→Serverless |
| **Monitoramento** | `@sentry/*`/`sentry-sdk`→Sentry · `dd-trace`/`datadog`→Datadog · `prometheus.yml`/`grafana/`→Grafana+Prometheus · `newrelic`→New Relic |
| **Registry de imagens** | Destino de `push` no workflow de CI: `ghcr.io`→GitHub Container Registry · `docker.io`→Docker Hub · `*.dkr.ecr.*`→AWS ECR · `*.pkg.dev`→GCP Artifact Registry |

---

## Comandos — onde achar os comandos reais

Prefira **sempre** os comandos declarados no projeto à derivação por tabela. Procure, em ordem:

| Fonte | Onde |
|-------|------|
| `package.json` | `scripts` (`dev`, `test`, `lint`, `typecheck`/`type-check`, `build`) |
| `composer.json` | `scripts` |
| `Makefile` | targets (`make dev`, `make test`, `make lint`, `make build`) |
| `Taskfile.yml` | tasks (`task <nome>`) |
| `justfile` | receitas (`just <nome>`) |
| `pyproject.toml` | `[tool.poetry.scripts]`, `[tool.pdm.scripts]`; `tox.ini`/`noxfile.py` |
| `Rakefile` | tasks Rake |
| `package.json` workspaces / `turbo.json` / `nx.json` | comandos de monorepo |

Mapeie cada script real para a ação correspondente (instalar, dev, test, lint, typecheck, build).
Só recorra às tabelas de `command-derivation.md` quando **não houver** scripts declarados — e, mesmo
assim, valide o comando contra o runtime detectado.

---

## Seleção de exemplos para os guias Backend (selecionar + normalizar)

> Usado na Fase 7 do `sdd.adopt` para substituir os exemplos curados de `api.md`, `services.md`,
> `tests.md` (e partes específicas de linguagem de `spec.md`) por exemplos do **código real** do
> projeto. Objetivo: guias fiéis ao projeto **sem enshrinar anti-patterns**.

**1. Localizar candidatos por tipo de exemplo:**

| Exemplo do guia | Onde procurar no projeto |
|-----------------|--------------------------|
| Controller / handler / rota | `controllers/`, `handlers/`, `routes/`, `app/Http/Controllers/`, `views.py`, `*_controller.rb` |
| Validação / request | `validators/`, `requests/`, schemas (`*.schema.*`, Pydantic models, FormRequest) |
| Middleware / auth | `middlewares/`, `middleware/`, guards, `app/Http/Middleware/` |
| Service / regra de negócio | `services/`, `app/Services/`, use-cases, `domain/` |
| Erros tipados | classes de erro/exception de domínio (`*Error`, `*Exception`) |
| Repository / acesso a dados | `repositories/`, `app/Repositories/`, DAOs |
| Teste unitário / integração | `tests/`, `__tests__/`, `*_test.go`, `*Test.php`, `spec/` |

**2. Critérios de seleção (evitar anti-patterns):**

- Escolha um arquivo **representativo** do padrão dominante, não o maior nem o mais excêntrico.
- Prefira código **coeso e legível**, que já segue a separação de camadas do guia.
- **Descarte** candidatos com cheiros óbvios (função gigante, lógica de negócio no controller,
  query crua no controller, segredos hardcoded). Se só houver candidatos ruins, prefira o **fallback**.

**3. Normalização (adaptar ao formato do guia):**

- Reduza ao **núcleo do padrão** — remova ruído irrelevante (imports longos, logging incidental,
  comentários mortos, detalhes de infra não essenciais).
- Mantenha nomes reais quando ajudam a ancorar; encurte corpos longos com `// ...` quando não.
- **Não corrija** silenciosamente um anti-pattern para "parecer bom" — se o trecho precisa de
  conserto para virar exemplo, ele **não é** um bom exemplo: use o fallback.
- Anote a origem no título do bloco: `### Exemplo: Service (baseado em app/services/user_service.py)`.

**4. Fallback (sem código representativo):** remova o bloco de exemplo daquela seção e deixe uma
nota curta — preservando sempre os **princípios/prosa** do guia:

```
> Exemplo omitido: não foi encontrado código representativo de <tipo> no projeto.
> Os princípios acima valem para <linguagem detectada>; adicione um exemplo conforme o código evoluir.
```

> **`spec.md` específico de linguagem:** ajuste a tabela de **Nomenclatura** e as seções
> "TypeScript (quando aplicável)" / "PHP (quando aplicável)" para a **linguagem detectada** —
> mantenha apenas a coluna/seção pertinente; se a linguagem for outra (Python, Go...), substitua
> pela convenção real do ecossistema (evidência: linters/formatters do projeto, ex: `ruff`,
> `gofmt`, `rubocop`).
