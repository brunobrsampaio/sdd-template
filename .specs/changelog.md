# Registro de Alterações

> Histórico de decisões que alteram princípios, guias de arquitetura ou convenções do projeto.
> Complementa o [`CLAUDE.md`](../CLAUDE.md) — não substitui os princípios em si.
> Remover esse bloco ao iniciar um novo projeto. O arquivo deve começar a partir da seção `Como usar`

---

> **Importante:** Este changelog registra decisões de design e arquitetura tomadas
> durante o desenvolvimento de features usando SDD. Ele **não** registra alterações
> nos arquivos de template (CLAUDE.md, guias de arquitetura, skills) — essas mudanças
> são versionadas normalmente via git. Use este arquivo quando uma decisão de projeto
> mudar princípios, stack, convenções ou direção técnica de forma deliberada.

---

## Como usar

Registre aqui mudanças **significativas** — não commits rotineiros de código. Use quando:

- Um artigo dos Princípios Não-Negociáveis for alterado ou removido
- Um guia em `.specs/architecture/` ganhar ou perder uma regra estrutural
- A stack ou convenção do projeto mudar de direção de forma deliberada

**Formato de cada entrada:**

| Data | Escopo | Mudança | Motivo |
|------|--------|---------|--------|
| [data] | [onde] | [o que mudou] | [por que mudou] |

**Valores comuns para Escopo:**

- **Princípios** — artigo do `CLAUDE.md` (ex: `Princípios — Artigo 2`)
- **Frontend** — `.specs/architecture/frontend/` ou guia complementar (`components.md`, `tests.md`, etc.)
- **Backend** — `.specs/architecture/backend/` ou guia complementar
- **Database** — `.specs/architecture/database/`
- **DevOps** — `.specs/architecture/devops/`
- **Geral** — estrutura de pastas, fluxo SDD, ferramentas adotadas

---

## Histórico

| Data | Escopo | Mudança | Motivo |
|------|--------|---------|--------|
| [data] | [ex: Princípios — Artigo 1] | [ex: unificação da constitution no CLAUDE.md] | [ex: template agnóstico de ferramenta SDD] |
