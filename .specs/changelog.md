# Registro de Alterações

> Histórico de decisões que alteram princípios, guias de arquitetura ou convenções do projeto.
> Complementa o [`CLAUDE.md`](../CLAUDE.md) — não substitui os princípios em si.
> Remover esse bloco ao iniciar um novo projeto. O arquivo deve começar a partir da seção `Como usar`

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
| 2026-06-08 | Geral — `CLAUDE.md` | Texto duplicado "specs Specs" corrigido; árvore de artefatos expandida com guias complementares | Inconsistência editorial; árvore incompleta omitia guias já existentes |
| 2026-06-08 | Frontend — `components.md` | `key={i}` → `key={String(row[rowKey])}` com prop `rowKey`; import de `User` adicionado no `AuthProvider` | Chave por índice causa bugs de reconciliação; import ausente tornava o exemplo inválido |
| 2026-06-08 | Geral — `README.md` | Reescrito: paths corrigidos, tabela alinhada à estrutura real, `sdd.setup` como único fluxo de setup | README desatualizado com paths errados e nomes incorretos |
| 2026-06-08 | Geral — `.gitignore` | Criado com `.DS_Store` | Arquivo de sistema macOS estava sendo rastreado pelo git |
| 2026-06-08 | Geral — `sdd.setup/SKILL.md` | Frontmatter: `model: opus`, `effort: high`, `disable-model-invocation: true` | Setup é operação única e crítica; Opus oferece melhor raciocínio para fluxo longo com lógica condicional |
