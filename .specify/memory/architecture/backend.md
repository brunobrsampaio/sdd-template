# Guia de Arquitetura — Backend

> Regras específicas para projetos com servidor, API ou serviços.
> Complementa a constitution — não a substitui.
> Seções marcadas com [PROJETO] devem ser ajustadas por projeto.
> Seções marcadas com [PADRÃO] refletem boas práticas gerais.
> Remover esse bloco ao iniciar um novo projeto. O arquivo deve começar a partir da seção `Stack`

---

## Stack [PROJETO]

- **Runtime:** [ex: Node.js 20]
- **Framework:** [ex: Fastify — proibido Express salvo justificativa]
- **Linguagem:** [ex: TypeScript — strict mode habilitado]
- **ORM / Query builder:** [ex: Drizzle ORM — proibido raw SQL salvo migrations]
- **Autenticação:** [ex: JWT com refresh token — proibido sessões em memória]
- **Validação:** [ex: Zod — validação em todas as entradas de API]

---

## Nomenclatura [PADRÃO]

- Arquivos: `kebab-case` (`user-profile.service.ts`)
- Classes e tipos: `PascalCase` (`UserProfileService`)
- Funções e variáveis: `camelCase` (`getUserProfile`)
- Constantes: `SCREAMING_SNAKE_CASE` (`MAX_LOGIN_ATTEMPTS`)
- Rotas: `kebab-case` em lowercase (`/user-profiles/:id`)
- Variáveis de ambiente: `SCREAMING_SNAKE_CASE` (`DATABASE_URL`)

---

## API Design [PADRÃO]

- Versionamento na URL: `/api/v1/...`
- Respostas de sucesso sempre com estrutura consistente:
  ```json
  { "data": {} }
  ```
- Respostas de erro sempre com estrutura consistente:
  ```json
  { "error": { "message": "..." } }
  ```
- Proibido retornar stack traces em produção
- Status HTTP usados semanticamente (200, 201, 400, 401, 403, 404, 422, 500)

---

## Segurança [PADRÃO]

- Toda entrada de API é validada antes de chegar na lógica de negócio
- Proibido logar dados sensíveis (senhas, tokens, CPF, cartão)
- Proibido expor IDs sequenciais em rotas públicas — use UUIDs
- Rate limiting em todas as rotas públicas e de autenticação
- Variáveis de ambiente para qualquer credencial — proibido hardcode

---

## Testes [PADRÃO]

- **Framework:** [ex: Jest ou Vitest]
- **HTTP mocking:** intercept na camada de repositório — não mockar o banco inteiro
- **Abordagem:** TDD — testes escritos antes ou junto com o código
- **Cobertura mínima:** 80% de branches na camada de serviço
- **Testes de integração:** rodar contra banco real em ambiente isolado (Docker)
- **O que testar:** lógica de negócio em serviços, contrato de rotas

---

## Tratamento de Erros [PADRÃO]

- Erros são tipados — classes de erro específicas por domínio (`UserNotFoundError`, `UnauthorizedError`)
- Handler global de erros converte exceções em respostas HTTP padronizadas
- Logs de erro incluem: rota, método, status, contexto relevante (sem dados sensíveis)
