# Guia de Arquitetura — Backend

> Regras específicas para projetos com servidor, API ou serviços.
> Complementa os princípios do projeto do `CLAUDE.md` — não os substitui.
> Seções marcadas com [PROJETO] devem ser ajustadas por projeto.
> Seções marcadas com [PADRÃO] refletem boas práticas gerais.
> Remover esse bloco ao iniciar um novo projeto. O arquivo deve começar a partir da seção `Stack`

---

## Stack [PROJETO]

- **Linguagem:** [ex: TypeScript strict | PHP 8.3]
- **Runtime:** [ex: Node.js 20 | PHP 8.3]
- **Framework:** [ex: Fastify | Laravel | NestJS | Symfony]
- **ORM / Query builder:** [ex: Drizzle ORM | Eloquent | Prisma | Doctrine]
- **Autenticação:** [ex: JWT com refresh token | Laravel Sanctum]
- **Validação:** [ex: Zod | Laravel Validator | class-validator]

---

## Qualidade e Padronização de Código [PADRÃO]

**A fonte da verdade absoluta para estilo e qualidade de código são os arquivos de configuração do projeto:**

- **Node.js/TS:** ESLint (`eslint.config.*`) + Prettier (`.prettierrc`)
- **PHP:** PHP-CS-Fixer (`.php-cs-fixer.php`) ou Pint (`pint.json`) + PHPStan (`phpstan.neon`)

Se esses arquivos existirem no repositório, **nenhuma regra deste guia pode contradizê-los**.
Em caso de conflito, o config do projeto vence — sempre.

As regras abaixo complementam os configs — servem para cobrir o que linters e formatadores não capturam.

### Nomenclatura

| Elemento | Node.js / TypeScript | PHP |
|----------|---------------------|-----|
| Arquivos | `kebab-case` (`user-profile.service.ts`) | `PascalCase` (`UserProfileService.php`) — PSR-4 |
| Classes/Tipos | `PascalCase` (`UserProfileService`) | `PascalCase` (`UserProfileService`) |
| Funções/Métodos | `camelCase` (`getUserProfile`) | `camelCase` (`getUserProfile`) |
| Constantes | `SCREAMING_SNAKE_CASE` (`MAX_RETRIES`) | `SCREAMING_SNAKE_CASE` (`MAX_RETRIES`) |
| Rotas | `kebab-case` lowercase (`/user-profiles/:id`) | `kebab-case` lowercase (`/user-profiles/{id}`) |
| Variáveis de ambiente | `SCREAMING_SNAKE_CASE` (`DATABASE_URL`) | `SCREAMING_SNAKE_CASE` (`DATABASE_URL`) |
| Interfaces/Types | `PascalCase` sem prefixo I (`UserProfile`) | `PascalCase` com sufixo Interface (`UserProfileInterface`) — PSR |

### Arquitetura de Camadas

Independente do framework, o backend segue separação em camadas com responsabilidades claras:

```
src/ (ou app/)
├── controllers/        # Recebe request, valida entrada, delega para service, retorna response
│   (ou handlers/)
├── services/           # Lógica de negócio — agnóstico de HTTP
├── repositories/       # Acesso a dados — único ponto de contato com banco/ORM
├── models/             # Entidades e definições de schema
│   (ou entities/)
├── middlewares/        # Autenticação, rate limiting, logging, validação
├── validators/         # Schemas de validação de entrada (Zod, class-validator, FormRequest)
│   (ou requests/)
├── types/              # Types/interfaces compartilhadas (TS) ou DTOs (PHP)
│   (ou dtos/)
└── utils/              # Funções utilitárias puras
    (ou helpers/)
```

> Nomes entre parênteses são alternativas comuns por ecossistema. Use a convenção do framework escolhido.

**Regras de dependência entre camadas:**
- **Controllers:** nunca acessam Repositories diretamente — sempre passam pelos Services
- **Services:** nunca importam objetos HTTP (Request/Response)
- **Repositories:** única camada que conhece o ORM/banco
- **Utils:** não dependem de nenhuma outra camada

### TypeScript (quando aplicável)

- **Strict mode:** `strict: true` habilitado — sem exceções
- **`any`:** proibido — use `unknown` quando o tipo é realmente indeterminado
- **Retorno:** explícito em funções públicas
- **DTOs/entidades:** sempre tipados

### PHP (quando aplicável)

- **Strict types:** `declare(strict_types=1)` em todo arquivo
- **Type hints:** em parâmetros e retornos — sem exceções
- **`mixed`:** proibido salvo em integrações com libs sem tipagem
- **Estilo:** PSR-12

---

## API Design [PADRÃO]

- **Versionamento:** na URL — `/api/v1/...`
- **Sucesso:** `{ "data": {} }`
- **Erro:** `{ "error": { "message": "...", "code": "..." } }`
- **Stack traces:** proibido retornar em produção
- **Status HTTP:** usados semanticamente (200, 201, 204, 400, 401, 403, 404, 409, 422, 500)
- **Paginação:** estrutura padronizada com envelope `data` + `meta` — `{ "data": [...], "meta": { "page", "per_page", "total", "last_page" } }`
- **Filtros/ordenação:** via query params (`?sort=name&order=asc&status=active`)

---

## Segurança [PADRÃO]

- **Validação de entrada:** toda entrada validada antes da lógica de negócio
- **Logs:** proibido logar dados sensíveis (senhas, tokens, CPF, cartão)
- **IDs:** sempre UUID — veja [`database/spec.md`](../database/spec.md)
- **Rate limiting:** em todas as rotas públicas e de autenticação
- **Credenciais:** variáveis de ambiente — proibido hardcode
- **CORS:** configurado explicitamente — proibido wildcard (`*`) em produção
- **Headers:** obrigatórios (X-Content-Type-Options, X-Frame-Options, etc.)

> Para regras de segurança em containers, CI e ambientes, veja [`devops/spec.md`](../devops/spec.md).

---

## Tratamento de Erros [PADRÃO]

- **Tipagem:** erros específicos por domínio (`UserNotFoundError`, etc.)
- **Handler global:** converte exceções em respostas HTTP padronizadas
- **Logs:** rota, método, status, contexto relevante (sem dados sensíveis)
- **Validação (422):** campo e mensagem por campo
- **Inesperado (500):** mensagem genérica ao cliente, log completo no servidor

---

## Performance [PADRÃO]

- **Queries:** índices apropriados — proibido full scan em tabelas grandes
- **Paginação:** obrigatória em listagens — proibido retornar coleções inteiras
- **Cache:** em camadas (HTTP, aplicação, banco) quando aplicável
- **Connection pooling:** configurado para o banco de dados

---

## Guias Complementares

Para detalhes, exemplos e anti-patterns de cada tema, consulte os arquivos dedicados nesta mesma pasta:

- **API:** [`api.md`](./api.md) — controllers, rotas, validação, middlewares e error handlers
- **Services:** [`services.md`](./services.md) — lógica de negócio, repositories e erros tipados
- **Testes:** [`tests.md`](./tests.md) — unitários, integração, factories e mocking
