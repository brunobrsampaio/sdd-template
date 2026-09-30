# Aplicação nos Arquivos

> Arquivo de referência compartilhado entre `sdd.setup`, `sdd.evolve` e `sdd.adopt`.
> Contém as instruções canônicas de edição de arquivos do projeto.
>
> **Uso no `sdd.setup`:** Aplicar todas as mudanças do zero (Fase de Aplicação).
> **Uso no `sdd.evolve`:** Aplicar mudanças incrementais — apenas o que foi adicionado
> ou modificado (Fase 9).
> **Uso no `sdd.adopt`:** Aplicar os valores **detectados** do projeto brownfield (Fase 7).
> Funciona como o `sdd.setup` (preenche todas as camadas presentes), com duas diferenças:
> os valores vêm da detecção (não de perguntas) e os exemplos dos guias do Backend são
> substituídos por trechos do código real — ver "Backend brownfield" abaixo.

---

## CLAUDE.md

### Identidade do Projeto

Preencher os campos da seção `Identidade do Projeto`:

- **Nome:** slug do projeto (ex: `strava-dashboard`)
- **Descrição curta:** 1 linha descrevendo o que é o projeto
- **Stack principal:** resumo das tecnologias escolhidas, separadas por ` + `
  (ex: `React 18 + TypeScript + Tailwind + shadcn/ui + Node.js 22 + Fastify + PostgreSQL`)

**No `sdd.setup`:** Remover o bloco de instruções no topo do arquivo (linhas com `>`
antes do primeiro `---`, incluindo a linha em branco seguinte) antes de preencher.

**No `sdd.evolve`:** Identidade do Projeto **não é alterada** — manter Nome e Descrição.
Atualizar apenas a Stack principal se novas tecnologias foram adicionadas ou modificadas.

**No `sdd.adopt`:** Preencher como no `sdd.setup`, mas a partir da detecção: Nome derivado do
diretório/manifesto do projeto (confirmado com o usuário), Descrição inferida do README/manifesto
(confirmada), e Stack principal composta com as tecnologias **detectadas** (inclusive valores fora
do catálogo, em texto livre).

### Regras de Composição da Stack Principal

A Stack principal é um resumo conciso das tecnologias centrais, no formato:
`<Frontend> + <Backend> + <Database>`

**Regras:**
1. **Frontend:** framework + estilo + UI library (se houver).
   Ex: `React 18 + TypeScript + Tailwind + shadcn/ui`
   Em modo Integrado: incluir a abordagem de integração.
   Ex: `Laravel + Inertia.js + React`

2. **Backend:** runtime + framework.
   Ex: `Node.js 22 + Fastify` ou `PHP 8.3 + Laravel`

3. **Database:** banco principal + ORM (se distinto do Backend).
   Ex: `PostgreSQL 16 + Drizzle ORM`

4. **Ordem:** Frontend → Backend → Database (sempre nesta sequência).

5. **Omissão:** Não incluir "Não se aplica". Não incluir DevOps ou ferramentas
   de suporte (CI/CD, monitoramento, etc.) — apenas stack de desenvolvimento.

6. **Máximo:** Manter em até 6 elementos. Se houver mais, priorizar os mais
   relevantes para o desenvolvimento diário.

**Exemplos:**
- `React 18 + TypeScript + Tailwind + Node.js 22 + Fastify + PostgreSQL`
- `Next.js 14 + Tailwind + shadcn/ui + PHP 8.3 + Laravel + MySQL`
- `Node.js 22 + NestJS + Drizzle ORM + PostgreSQL` (sem frontend)

### Guias de Arquitetura Ativos

Marcar `[x]` nos guias que se aplicam e `[ ]` nos demais:

```markdown
- [x] `.specs/architecture/frontend/spec.md`
- [x] `.specs/architecture/backend/spec.md`
- [ ] `.specs/architecture/database/spec.md`
- [ ] `.specs/architecture/devops/spec.md`
```

**No `sdd.setup`:** Marcar `[x]` em todos os guias selecionados na Fase 2.

**No `sdd.evolve`:** Adicionar `[x]` nas novas camadas. Camadas já ativas permanecem
com `[x]`. Camadas não ativas permanecem com `[ ]`.

**No `sdd.adopt`:** Marcar `[x]` em cada camada **detectada como presente** no projeto
(ver "Detecção de camadas presentes" em `stack-detection.md`); as demais ficam `[ ]`.

### Comandos do Projeto

Substituir a linha placeholder pela tabela de comandos derivados:

```markdown
| Ação | Comando | Camada |
|------|---------|--------|
| Instalar dependências | `composer install` | backend |
| Instalar dependências | `npm install` | frontend |
| Servidor de dev | `php artisan serve` | backend |
| ... | ... | ... |
```

Cada comando em sua própria linha. A coluna "Camada" identifica a qual camada pertence
(frontend, backend, devops, etc.). Omita ações que não se aplicam.

### Referências Rápidas

Adicionar os links informados pelo usuário. Se não houver links, remover o placeholder
da seção.

**No `sdd.evolve`:** Manter links existentes, adicionar novos, remover os que o usuário
solicitar.

---

## Specs de Arquitetura — spec.md

### Procedimento padrão (qualquer camada)

Para cada guia ativo:

1. **Remover bloco de instruções:** Remover as linhas com `>` antes do primeiro `---`
   (incluindo a linha em branco seguinte).

2. **Preencher `## Stack [PROJETO]`:** O título permanece `## Stack [PROJETO]` (a tag não é removida).
   Preencher com as respostas do questionário. Exemplo para Frontend:
   ```markdown
   ## Stack [PROJETO]
   
   - **Framework:** React 18 com TypeScript — strict mode habilitado
   - **Estilização:** Tailwind CSS — proibido CSS-in-JS e styled-components
   - **Estado global:** Zustand — proibido Redux
   - **Estado de servidor:** TanStack Query — proibido fetch manual em useEffect para dados remotos
   - **Formulários:** React Hook Form — proibido Formik
   - **Roteamento:** React Router v6
   - **UI Library:** shadcn/ui — proibido usar componentes não acessíveis
   ```

3. **Remover campos "não se aplica":** Se o usuário indicou "Não se aplica" para um campo,
   remover a linha correspondente da seção `## Stack [PROJETO]`.

> **No `sdd.adopt`:** mesmo procedimento, com os valores vindos da **detecção** (`stack-detection.md`).
> A seção `## Stack [PROJETO]` aceita **valores livres** — preencha com o nome real da tecnologia
> mesmo quando ela não existe no catálogo do `question-flows.md` (ex: `Vue 3`, `Django 5`, `GORM`).
> **Importante:** o passo 3 é **diferente** no `sdd.adopt` — campos **não detectados** não são
> removidos; permanecem com o valor `A definir` como placeholder para documentação futura.
> Nenhum campo da seção Stack é removido, independente de ter sido detectado ou não.

### Procedimento para o `sdd.evolve` (modificação)

Para camadas **modificadas** (não adicionadas do zero):

1. **Apenas campos alterados:** Atualizar somente as linhas cujo valor mudou.
   Campos mantidos (usuário escolheu "Manter: ...") **não são alterados**.

2. **Campos removidos:** Se o usuário escolheu "Não se aplica" para um campo que antes
   tinha valor, remover a linha correspondente.

3. **Campos adicionados:** Se o usuário escolheu um valor para um campo que antes era
   "Não se aplica", adicionar a linha correspondente.

### Guias inativos

Não modificar guias que não foram selecionados como ativos.

---

## Backend — Arquivos Complementares

Os arquivos `tests.md`, `services.md` e `api.md` em `.specs/architecture/backend/`
contêm exemplos nas duas linguagens (Node.js/TypeScript e PHP). Após identificar a
linguagem escolhida, **remover os exemplos e referências da linguagem não selecionada**.

### Se a linguagem é TypeScript / JavaScript

**Em `tests.md`:**
- Remover: "Exemplo: Teste unitário de service (PHP/Laravel)"
- Remover: "Exemplo: Teste de integração de rota (PHP/Laravel)"

**Em `services.md`:**
- Remover: "Exemplo: Service (PHP/Laravel)"
- Remover: "Exemplo: Erros tipados (PHP/Laravel)"

**Em `api.md`:**
- Remover: "Exemplo: Controller (PHP/Laravel)"
- Remover: "Exemplo: Validação (PHP/Laravel — FormRequest)"
- Remover: "Exemplo: Error handler global (PHP/Laravel)"

**Em `spec.md`:**
- Na tabela de Nomenclatura, remover a coluna PHP
- Remover a seção "PHP (quando aplicável)" inteiramente

### Se a linguagem é PHP

**Em `tests.md`:**
- Remover: "Exemplo: Teste unitário de service (Node.js/TypeScript)"
- Remover: "Exemplo: Teste de integração de rota (Node.js/TypeScript)"
- Remover: "Exemplo: Factory para testes"

**Em `services.md`:**
- Remover: "Exemplo: Service (Node.js/TypeScript)"
- Remover: "Exemplo: Erros tipados (Node.js/TypeScript)"
- Remover: "Exemplo: Repository (Node.js/TypeScript)"

**Em `api.md`:**
- Remover: "Exemplo: Controller (Node.js/TypeScript — Fastify)"
- Remover: "Exemplo: Validação (Node.js/TypeScript — Zod)"
- Remover: "Exemplo: Middleware de autenticação (Node.js/TypeScript)"
- Remover: "Exemplo: Error handler global (Node.js/TypeScript)"

**Em `spec.md`:**
- Na tabela de Nomenclatura, remover a coluna "Node.js / TypeScript"
- Remover a seção "TypeScript (quando aplicável)" inteiramente

### Em ambos os casos

- Nos anti-patterns, manter apenas os exemplos na linguagem selecionada. Se um anti-pattern
  tem exemplos nas duas linguagens, remover o da linguagem não selecionada.
- Atualizar as tabelas comparativas em `spec.md` (seção Testes) para manter apenas a linha
  da linguagem selecionada.

### No `sdd.evolve`

A linguagem do Backend é um campo **fundamental** — não muda numa modificação. Esta limpeza só
se aplica ao **adicionar** um Backend novo: execute a seção correspondente acima conforme a
linguagem escolhida na adição. Camadas Backend já existentes não têm os arquivos complementares
re-processados.

### No `sdd.adopt` — Backend brownfield: exemplos a partir do código real

> Substitui o procedimento de limpeza acima quando o Backend vem de um projeto **existente**.
> Em vez de só remover os exemplos da linguagem não usada, **troca** os exemplos curados por
> exemplos extraídos do **código real do projeto**. A mecânica de seleção e normalização é
> canônica em **"Seleção de exemplos para os guias Backend"** de `stack-detection.md`.

Para cada bloco de exemplo em `api.md`, `services.md` e `tests.md` (controller/handler, validação,
middleware/auth, service, erros tipados, repository, testes):

1. **Localizar** código representativo do projeto para aquele tipo (ver tabela em `stack-detection.md`).
2. **Selecionar** um exemplar limpo e representativo — descartar candidatos com anti-patterns óbvios.
3. **Normalizar** ao formato do guia (reduzir ao núcleo do padrão, remover ruído) **sem corrigir**
   anti-patterns para "parecer bom".
4. **Substituir** o bloco curado pelo exemplo normalizado e **anotar a origem** no título:
   `### Exemplo: Service (baseado em app/services/user_service.py)`.
5. **Fallback** (sem código representativo): remover aquele bloco e deixar a nota curta de
   `stack-detection.md` — preservando sempre os **princípios/prosa** do guia.

**Em `spec.md`:** ajustar a tabela de **Nomenclatura** e as seções "TypeScript (quando aplicável)" /
"PHP (quando aplicável)" para a **linguagem detectada**:
- Linguagem TS/JS ou PHP → manter apenas a coluna/seção correspondente (como na limpeza padrão).
- Linguagem fora do catálogo (Python, Go, Ruby...) → substituir pela convenção real do ecossistema,
  ancorada na evidência do projeto (linters/formatters, ex: `ruff`, `gofmt`, `rubocop`).

> **Princípios e prosa (API Design, Segurança, Tratamento de Erros, Performance) nunca são
> removidos** — são agnósticos de linguagem. Só os blocos de **exemplo de código** são trocados.

---

## Validação Pós-Aplicação

> Checklist canônico executado **antes da confirmação final**, nos três skills.
> Itens marcados `[setup]` valem só para `sdd.setup`; `[evolve]` só para `sdd.evolve`;
> `[adopt]` só para `sdd.adopt`; os demais valem para todos.

1. **Placeholders `[ex: ...]`:** Nenhum arquivo modificado deve conter placeholders
   `[ex: ...]` remanescentes. Se houver, identifique o motivo e preencha.

2. **Stack populada:** Cada `spec.md` de guia ativo deve ter a seção `## Stack [PROJETO]`
   com todos os campos não-"Não se aplica" preenchidos — sem placeholders.
   - `[evolve]` Em camadas **modificadas**, apenas campos alterados foram atualizados;
     campos mantidos ("Manter: ...") permanecem com os valores originais.
   - `[adopt]` Os valores refletem a **detecção** (incluindo valores livres fora do catálogo);
     campos de baixa confiança/não detectados foram confirmados com o usuário; campos não
     detectados permanecem como `A definir` (placeholder para documentação futura).

3. **Limpeza de linguagem (Backend):** Os arquivos `api.md`, `services.md`, `tests.md` e
   `spec.md` do Backend não devem conter exemplos da linguagem não selecionada.
   `[evolve]` Aplicável quando um Backend é **adicionado** (a linguagem não muda em modificação).
   - `[adopt]` Em vez disso, os exemplos foram **substituídos por trechos do código real** do
     projeto (anotados com a origem) ou removidos com nota quando não havia código representativo;
     os princípios/prosa dos guias permanecem intactos.

4. **Sincronização cross-layer `[evolve]`:** ORM consistente entre Backend e Database, se
   o usuário optou por sincronizar.

5. **Tabela de comandos:** O `CLAUDE.md` deve ter a tabela populada, sem placeholder
   `[ex: ...]`, com coluna "Camada" identificando cada comando.
   - `[evolve]` Comandos novos/modificados presentes; comandos de camadas não alteradas preservados.

6. **Integridade de links:** Links do `CLAUDE.md` para `.specs/` devem apontar para
   arquivos que realmente existem.

---

## Formato de Resumo (Confirmação)

### Template do `sdd.setup`

```
## Resumo do projeto

**Nome:** <slug>
**Descrição:** <descrição>
**Guias ativos:** <lista dos guias selecionados>
**Modo de arquitetura:** <Separados | Integrado — omitir se apenas uma camada ativa>

### Frontend
- Framework: <valor>
- Estilização: <valor>
- Estado global: <valor>
- Estado de servidor: <valor>
- Formulários: <valor>
- Roteamento: <valor>
- UI Library: <valor>

### Backend
- Linguagem: <valor>
- Runtime: <valor>
- Framework: <valor>
- ORM / Query builder: <valor>
- Autenticação: <valor>
- Validação: <valor>

### Database
- Banco principal: <valor>
- ORM / Query builder: <valor>
- Migrations: <valor>
- Banco de desenvolvimento: <valor>
- Banco de teste: <valor>

### DevOps
- CI/CD: <valor>
- Containerização: <valor>
- Hospedagem: <valor>
- Monitoramento: <valor>
- Registry de imagens: <valor>

### Comandos
<lista de comandos derivados, um por linha>

### Referências
<links informados, ou "Nenhuma" se não houver>
```

> Omitir seções de camadas não selecionadas.

### Template do `sdd.evolve`

Mesmo formato acima, mas com indicadores visuais de mudança:

- `⚡` — campo alterado (ex: `Framework: React 18 → Next.js 14  ⚡`)
- `✨` — camada ou campo novo
- `(mantido)` — campo não alterado
- `[NOVO]` — camada adicionada
- `[ativo — modificado]` — camada existente com alterações
- `[ativo — mantido]` — camada existente sem alterações

Exemplo de resumo de comandos com indicadores:
```
+ npm install (backend)           — NOVO
+ npm run dev (backend)           — NOVO
  npm install (frontend)          — mantido
  npm run dev (frontend)          — mantido
```

### Template do `sdd.adopt`

Mesmo formato do `sdd.setup`, com a **origem de cada valor** anexada (confiança + evidência),
para o usuário revisar a detecção:

```
### Backend (detectado)
- Linguagem: TypeScript            · alta   · package.json + tsconfig.json
- Framework: Fastify               · alta   · package.json:24
- ORM / Query builder: Drizzle ORM · alta   · drizzle.config.ts
- Autenticação: JWT                · média  · @fastify/jwt em package.json
- Validação: (não detectado)       · —      · confirmar com o usuário
```

- **detectado** — valor de confiança **alta**, apresentado para o usuário só confirmar.
- **média / baixa** — apresentado, mas sinalizado para revisão.
- **(não detectado)** — campo sem evidência → entra no preenchimento de lacunas.

Liste também os **exemplos do Backend** que serão substituídos e sua origem
(ex: `services.md ← app/services/user_service.ts`) ou marcados para remoção (sem código).

### Pergunta de confirmação (os três skills)

```
questions: [
  {
    question: "As informações acima estão corretas?",
    header: "Confirmação",
    options: [
      { label: "Confirmar e aplicar", description: "Prosseguir com a configuração dos arquivos" },
      { label: "Reiniciar",           description: "Descartar as respostas e refazer o questionário desde o início" }
    ]
  }
]
```
