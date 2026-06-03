# Guia de Arquitetura — DevOps

> Regras específicas para CI/CD, ambientes, containers e infraestrutura.
> Complementa os princípios não-negociáveis do `CLAUDE.md` — não os substitui.
> Seções marcadas com [PROJETO] devem ser ajustadas por projeto.
> Seções marcadas com [PADRÃO] refletem boas práticas gerais.
> Remover esse bloco ao iniciar um novo projeto. O arquivo deve começar a partir da seção `Stack`

---

## Stack [PROJETO]

- **CI/CD:** [ex: GitHub Actions]
- **Containerização:** [ex: Docker + Docker Compose para desenvolvimento local]
- **Hospedagem:** [ex: Railway para API, Vercel para frontend]
- **Monitoramento:** [ex: Sentry para erros, Grafana para métricas]
- **Registry de imagens:** [ex: GitHub Container Registry]

---

## Ambientes [PADRÃO]

```
local       ← local, dados fictícios, logs verbosos
staging     ← espelho de produção, dados anonimizados
production  ← real, monitorado, acesso restrito
```

- Proibido testar diretamente em produção
- Staging deve refletir produção em configuração — não em dados
- Variáveis de ambiente são gerenciadas por ambiente — nunca compartilhadas entre eles

---

## Variáveis de Ambiente [PADRÃO]

- Toda credencial, URL de serviço externo e configuração sensível é variável de ambiente
- Proibido commitar arquivos `.env` com valores reais — apenas `.env.example` com chaves vazias
- Nomenclatura: `SCREAMING_SNAKE_CASE`, prefixada pelo domínio (`DATABASE_URL`, `STRIPE_SECRET_KEY`)
- Documentar toda variável nova no `.env.example` com comentário explicativo

```bash
# .env.example
DATABASE_URL=          # URL de conexão PostgreSQL
STRIPE_SECRET_KEY=     # Chave secreta da API do Stripe
JWT_SECRET=            # Secret para assinatura de tokens JWT
```

---

## CI Pipeline [PADRÃO]

Todo PR deve passar obrigatoriamente por:

1. **Lint** — proibido merge com erros de lint
2. **Typecheck** — proibido merge com erros de tipo
3. **Testes** — proibido merge com testes falhando
4. **Build** — proibido merge se o build quebrar

```yaml
# Ordem obrigatória no pipeline
lint → typecheck → test → build → deploy (apenas na branch principal)
```

---

## Docker [PADRÃO]

- Imagens de produção baseadas em imagens `slim` ou `alpine` — sem ferramentas de desenvolvimento
- Multi-stage build: estágio de build separado do estágio de execução
- Proibido rodar container como root em produção — use usuário não-privilegiado
- `.dockerignore` configurado para excluir `node_modules`, `.env`, arquivos de teste e docs

```dockerfile
# Estrutura padrão multi-stage
FROM node:20-alpine AS builder
# ... build

FROM node:20-alpine AS runner
# ... apenas o necessário para rodar
```

---

## Branching [PADRÃO]

```
main          ← produção, protegida, só via PR
staging       ← ambiente de staging, merge de feature branches
feat/<nome>   ← feature em desenvolvimento
fix/<nome>    ← correção de bug
chore/<nome>  ← tarefas de manutenção (deps, config, etc.)
```

- PRs para `main` requerem ao menos 1 aprovação
- Proibido commit direto em `main` e `staging`
- Branch deletada após merge

---

## Secrets e Segurança [PADRÃO]

- Secrets do CI armazenados no cofre da plataforma (GitHub Secrets, etc.) — nunca em código
- Proibido logar variáveis de ambiente em pipelines de CI
- Dependências auditadas em cada PR (`npm audit` ou equivalente)
- Imagens Docker escaneadas por vulnerabilidades antes do deploy em produção
