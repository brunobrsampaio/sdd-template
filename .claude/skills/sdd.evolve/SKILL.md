---
name: sdd.evolve
description: Evolui a configuração de um projeto SDD — adiciona novas camadas de arquitetura, modifica stacks existentes ou ambas. Use após o sdd.setup, quando o projeto crescer e precisar de novas camadas ou alterações nas escolhas atuais.
disable-model-invocation: true
model: opus
effort: high
---

## O que esse skill faz

Permite evoluir um projeto que já passou pelo `sdd.setup`, adicionando novas camadas de
arquitetura (Frontend, Backend, Database, DevOps) ou modificando as escolhas de stack das
camadas existentes — sempre preservando o que já está configurado.

Diferente do `sdd.setup`, que parte de um estado vazio, o `sdd.evolve`:

- Detecta o estado atual do projeto (guias ativos, stacks, comandos)
- Permite adicionar camadas que não existiam
- Permite modificar camadas existentes com a opção "Manter: <valor atual>" em cada campo
- Sincroniza dependências cross-layer (ex: ORM do Backend com Database)
- Atualiza comandos e stacks de forma incremental

Antes de começar, execute a **Fase 0** para detectar o estado atual do projeto.

---

## Fase 0 — Detecção do Estado Atual

Leia os seguintes arquivos para extrair o estado atual do projeto:

### 1. CLAUDE.md

- **Nome:** valor do campo `Nome` na seção `Identidade do Projeto`
- **Descrição:** valor do campo `Descrição curta`
- **Stack principal:** valor do campo `Stack principal`
- **Guias ativos:** quais linhas têm `[x]` na seção `Guias de Arquitetura Ativos`
- **Comandos:** tabela completa na seção `Comandos do Projeto`
- **Referências:** links na seção `Referências Rápidas`

### 2. Specs ativas

Para cada guia marcado com `[x]`, leia o `spec.md` correspondente e extraia os valores da
seção `## Stack [PROJETO]`:

**Frontend** (`.specs/architecture/frontend/spec.md`):
- Framework, Estilização, Estado global, Estado de servidor, Formulários, Roteamento, UI Library

**Backend** (`.specs/architecture/backend/spec.md`):
- Linguagem, Runtime, Framework, ORM / Query builder, Autenticação, Validação

**Database** (`.specs/architecture/database/spec.md`):
- Banco principal, ORM / Query builder, Migrations, Banco de desenvolvimento, Banco de teste

**DevOps** (`.specs/architecture/devops/spec.md`):
- CI/CD, Containerização, Hospedagem, Monitoramento, Registry de imagens

### 3. Detecção do modo de arquitetura

Se **Frontend e Backend** estão ambos ativos, determine o modo (Separados ou Integrado):

- Se o `CLAUDE.md` tem comandos separados por camada com seus próprios servidores de dev → **Separados**
- Se há referências a Inertia.js, Livewire, Blade, Twig, HTMX ou templates no Frontend → **Integrado**
- Se o Frontend usa "Não se aplica" para roteamento e o backend o serve → **Integrado**
- Se nenhum sinal acima permitir determinar o modo com confiança, **não assuma um padrão** —
  pergunte ao usuário usando o bloco `AskUserQuestion` da seção **"Modo de Arquitetura"** de
  `.claude/skills/sdd.references/question-flows.md`.

### 4. Validação inicial

Se o `CLAUDE.md` **ou** qualquer `spec.md` de guia ativo (marcado `[x]`) ainda contiver
placeholders `[ex: ...]` (ex: `[ex: React 18 com TypeScript...]`), **recuse-se a continuar** com a
seguinte mensagem:

> O projeto ainda não foi configurado pelo `sdd.setup`. Execute a skill `sdd.setup` primeiro
> para preencher os placeholders `[ex: ...]` e depois use `sdd.evolve` para evoluir a configuração.

Se a validação passar, apresente um resumo do estado atual ao usuário antes de prosseguir:

```
## Estado atual do projeto

**Nome:** <slug>
**Stack:** <stack principal>
**Guias ativos:** <lista dos ativos>

### Frontend (ativo)
- Framework: <valor>
- Estilização: <valor>
- ...

### Backend (ativo)
- ...

(omitir camadas inativas)
```

> Ao exibir o estado, marque os campos fundamentais — Framework (Frontend); Linguagem, Runtime
> e Framework (Backend); Banco principal (Database) — como `(base — não alterável via evolve)`.
> Eles não serão oferecidos para modificação (ver Fase 4B).

---

## Fase 1 — Tipo de Evolução

Use `AskUserQuestion` para definir o que o usuário deseja fazer:

```
questions: [
  {
    question: "O que você deseja fazer?",
    header: "Evolução",
    multiSelect: true,
    options: [
      {
        label: "Adicionar camada(s)",
        description: "Ativar Frontend, Backend, Database ou DevOps que não existem hoje"
      },
      {
        label: "Modificar camada(s) existente(s)",
        description: "Alterar stack de camadas já ativas — framework, ORM, bibliotecas, etc."
      },
      {
        label: "Ambos",
        description: "Adicionar novas camadas E modificar as existentes em uma única sessão"
      }
    ]
  }
]
```

- Se `Adicionar camada(s)`: prossiga para a Fase 2A
- Se `Modificar camada(s) existente(s)`: prossiga para a Fase 2B
- Se `Ambos`: execute a Fase 2A primeiro, depois a Fase 2B

---

## Fase 2A — Seleção de Camadas para Adicionar

> Executar se o usuário escolheu "Adicionar camada(s)" ou "Ambos" na Fase 1.

Liste apenas as camadas **não ativas** atualmente. Se todas as camadas já estiverem ativas,
informe o usuário e pule para a Fase 2B (se "Ambos") ou encerre.

```
questions: [
  {
    question: "Quais camadas você deseja adicionar ao projeto?",
    header: "Adicionar",
    multiSelect: true,
    options: [
      // Apenas camadas NÃO ativas — filtre dinamicamente
      { label: "Frontend",  description: "Interface web: SPA ou SSR" },
      { label: "Backend",   description: "Servidor, API REST/GraphQL ou serviços" },
      { label: "Database",  description: "Banco de dados (relacional ou NoSQL)" },
      { label: "DevOps",    description: "CI/CD, containers, deploy e infraestrutura" }
    ]
  }
]
```

---

## Fase 2B — Seleção de Camadas para Modificar

> Executar se o usuário escolheu "Modificar camada(s) existente(s)" ou "Ambos" na Fase 1.

Liste apenas as camadas **ativas** atualmente. A `description` de cada opção deve conter um
resumo compacto da stack atual daquela camada, extraído na Fase 0.

Exemplo de montagem das descriptions:

- **Frontend:** `Atual: React 18 + Tailwind + Zustand + TanStack Query`
- **Backend:** `Atual: Node.js 22 + Fastify + Drizzle ORM + JWT`
- **Database:** `Atual: PostgreSQL 16 + Drizzle ORM + Docker local`
- **DevOps:** `Atual: GitHub Actions + Docker Compose + Railway`

```
questions: [
  {
    question: "Quais camadas você deseja modificar?",
    header: "Modificar",
    multiSelect: true,
    options: [
      // Apenas camadas ATIVAS — filtre dinamicamente com descriptions do estado atual
      { label: "Frontend",  description: "Atual: <resumo da stack atual>" },
      { label: "Backend",   description: "Atual: <resumo da stack atual>" },
      { label: "Database",  description: "Atual: <resumo da stack atual>" },
      { label: "DevOps",    description: "Atual: <resumo da stack atual>" }
    ]
  }
]
```

Se nenhuma camada estiver ativa, informe e encerre.

> **Campos fundamentais não são modificáveis.** Selecionar uma camada aqui significa alterar
> seus campos **modificáveis** (ORM, autenticação, validação, UI, estilização, migrations, DevOps).
> Framework do Frontend, Linguagem/Runtime/Framework do Backend e o Banco principal ficam de
> fora — ver Fase 4B.

---

## Fase 3 — Modo de Arquitetura (condicional)

**Dispara apenas quando**, após a Fase 2, o projeto passará a ter **Frontend E Backend
simultaneamente** e essa combinação não existia antes.

**Gatilhos:**
- Projeto tinha só Frontend e adicionou Backend
- Projeto tinha só Backend e adicionou Frontend

Se disparar, use o bloco `AskUserQuestion` da seção **"Modo de Arquitetura"** de
`.claude/skills/sdd.references/question-flows.md`.

- Se **Separados**: prossiga normalmente — Frontend e Backend são tratados de forma independente.
- Se **Integrado**: na configuração das camadas, execute a seção **Backend** primeiro e, ao
  terminar, use a seção **Frontend Integrado** do `question-flows.md`. Pule a seção Frontend padrão.

Se Frontend e Backend **já coexistiam**, pule esta fase — o modo atual é preservado.

---

## Fase 4 — Configuração das Camadas

Para cada camada selecionada na Fase 2, execute o fluxo apropriado:

### 4A — Camadas NOVAS (adição)

> Leia `.claude/skills/sdd.references/addition-flow.md` — fonte canônica da sequência de
> adição (ordem de execução, passos por camada e combinação de campos). As opções de cada
> campo estão em `.claude/skills/sdd.references/question-flows.md`.

Para cada camada **nova**, execute o fluxo da seção correspondente em `addition-flow.md`.
A ordem entre camadas (Backend → Database → Frontend → DevOps), o ramo do modo Integrado e a
herança de ORM do Database já estão descritos lá. Aplique por cima os ajustes específicos
de evolução abaixo.

**Ajuste de evolução — Backend novo com Database já ativo:**
Se o ORM escolhido no Backend diferir do ORM de um Database **já ativo**, **não pergunte agora** —
apenas registre o ORM do Backend. A reconciliação acontece num único ponto, na **Fase 5.1**.
(Se o Database também é novo nesta sessão, ele herda o ORM do Backend automaticamente, conforme
`addition-flow.md` > Database.)

---

### 4B — Camadas EXISTENTES (modificação)

> Aplique o **Padrão "Manter"** documentado em `.claude/skills/sdd.references/modification-pattern.md`.
> Para cada campo, use as opções da seção correspondente em `question-flows.md` conforme o
> mapeamento no arquivo de referência.
>
> Consulte a **Tabela de Cascata** no `modification-pattern.md` para saber quais campos
> reavaliar após uma alteração.

**Campos fundamentais não são modificáveis.** Não pergunte (nem ofereça "Manter") sobre
Framework do Frontend, Linguagem/Runtime/Framework do Backend ou Banco principal. Se o usuário
pedir para trocar um deles, use a **saída de emergência** da seção "Campos Fundamentais (não
modificáveis)" do `modification-pattern.md`. Apenas os campos **modificáveis** entram no fluxo abaixo.

Vale a regra per-camada do `modification-pattern.md` ("O Padrão Manter"): se o usuário
escolher "Manter: ..." em todos os campos **modificáveis** de uma camada, ela não sofre alteração —
pule a edição de arquivos para ela.

Para cada camada selecionada, faça **uma chamada `AskUserQuestion` por campo modificável**, seguindo o
padrão "Manter". A **ordem dos campos** de cada camada e os **ajustes específicos de modificação**
(UI Library condicional, herança de ORM do Database) estão em `modification-pattern.md` >
"Mapeamento: Campo → question-flows.md". A seção e o `header` de cada campo vêm da tabela
"Mapeamento Rápido" de `question-flows.md`.

> ⚠️ **Campos escopados por linguagem** (ORM, Autenticação, Validação, Migrations): as alternativas
> saem da variante (`— PHP` / `— TS/JS`) que **contém o valor atual** — `Manter: Eloquent` ⇒ opções
> PHP (Doctrine, Cycle ORM), **nunca** Drizzle/Prisma. Ver regra 6 ("Coerência de ecossistema") em
> `modification-pattern.md`.

---

## Fase 5 — Sincronização Cross-Layer

Após todas as alterações da Fase 4, verifique inconsistências entre camadas:

### 5.1 — ORM: Backend vs Database

> **Ponto único de reconciliação de ORM.** A Fase 4 nunca pergunta isso — apenas registra os
> ORMs. Toda divergência Backend × Database é resolvida aqui, uma única vez.

Se Backend e Database estão ambos ativos e os ORMs são diferentes:

```
questions: [
  {
    question: "O Backend usa <ORM backend> mas o Database está com <ORM database>. Qual ORM deve ser usado no Database?",
    header: "Sincronizar ORM",
    options: [
      { label: "Usar <ORM backend>", description: "Alinhar o Database ao ORM do Backend" },
      { label: "Manter <ORM database>", description: "Manter o ORM atual do Database — os ORMs serão diferentes" }
    ]
  }
]
```

### 5.2 — Migrations vs ORM

Se a ferramenta de migration do Database for incompatível com o ORM (ex: Drizzle ORM com
Prisma Migrate), alerte e ofereça correção:

```
questions: [
  {
    question: "A ferramenta de migration (<migration atual>) não é compatível com o ORM (<ORM atual>). Deseja ajustar?",
    header: "Migration vs ORM",
    options: [
      { label: "<migration compatível>", description: "Ferramenta compatível com <ORM atual>" },
      { label: "Manter assim", description: "Manter a configuração atual mesmo com incompatibilidade" }
    ]
  }
]
```

### 5.3 — Modo Integrado: consistência

Se Frontend e Backend existem no modo Integrado, mas a stack do Frontend não reflete isso
(ex: tem roteamento próprio quando deveria ser "Não se aplica"), alerte o usuário.

---

## Fase 6 — Comandos do Projeto

> Leia `.claude/skills/sdd.references/command-derivation.md` para as tabelas canônicas
> de derivação de comandos.

Re-derive os comandos usando as tabelas do arquivo de referência, com as seguintes
particularidades:

**Regras de preservação:**
- Comandos de camadas **não alteradas** (nem adicionadas nem modificadas): **mantenha como estão**
- Comandos de camadas **novas**: derive do zero usando as tabelas do arquivo de referência
- Comandos de camadas **modificadas**: re-derive apenas se a mudança afetar comandos:
  - Mudar ORM, autenticação, validação, migrations → **não** afeta comandos
  - Mudar containerização (DevOps) → **afeta** comandos (re-derive)
  - (Framework, runtime e linguagem são fundamentais — não mudam numa modificação.)

**Conflito Docker + servidores de dev:**
Use a mesma lógica da seção "DevOps / Docker" do arquivo de referência, mas considere
que o estado anterior pode já ter resolvido esse conflito. Se a configuração de
containers ou servidores de dev não mudou, mantenha o comando existente.

Apresente a lista final de comandos ao usuário em texto livre para confirmação.
Formato: uma tabela ou lista com ação, comando e camada.

---

## Fase 7 — Referências Rápidas

Apresente os links atuais (extraídos na Fase 0) e pergunte em texto livre:

> Esses são os links atuais nas Referências Rápidas:
> - `<link1>`
> - `<link2>`
>
> Deseja adicionar novos links ou remover algum existente?

Se o usuário quiser modificar, aceite as alterações em texto livre.
Se não houver alterações, mantenha o que existe.

Não use `AskUserQuestion` nesta fase.

---

## Fase 8 — Confirmação e Resumo

Exiba um resumo completo de tudo que será alterado, usando o **"Template do `sdd.evolve`"**
da seção "Formato de Resumo (Confirmação)" de
`.claude/skills/sdd.references/file-application.md` — com os indicadores visuais de mudança
(`⚡` alterado, `✨` novo, `(mantido)`, `[NOVO]`) definidos lá.

Em seguida, use a **"Pergunta de confirmação"** canônica da mesma seção do arquivo de referência.

Se **Reiniciar**, volte ao início da Fase 1.
Se **Confirmar e aplicar**, prossiga para a Fase 9.

---

## Fase 9 — Aplicação das Mudanças

> Leia `.claude/skills/sdd.references/file-application.md` para as instruções canônicas
> de edição de cada arquivo.

### 9.1 — CLAUDE.md

Aplique as instruções da seção "CLAUDE.md" do arquivo de referência, com as adaptações
para evolução:
1. **Guias de Arquitetura Ativos:** Marcar `[x]` nas novas camadas; manter `[x]` nas existentes
2. **Stack principal:** Atualizar com tecnologias das novas camadas e alterações
3. **Comandos do Projeto:** Substituir pela nova tabela derivada na Fase 6
4. **Identidade do Projeto:** Manter inalterada (Nome, Descrição curta)
5. **Referências Rápidas:** Atualizar com adições/remoções da Fase 7

### 9.2 — Specs de camadas NOVAS

Aplique as instruções da seção "Specs de Arquitetura — spec.md" do arquivo de referência.
Para Backend novo, aplique também a seção "Backend — Arquivos Complementares" conforme
a linguagem escolhida.

### 9.3 — Specs de camadas MODIFICADAS

Aplique as instruções da seção "Specs de Arquitetura — spec.md" do arquivo de referência,
com as seguintes adaptações:
1. **Apenas campos alterados:** Atualizar somente linhas cujo valor mudou; campos "Manter:" não são alterados
2. **Campos removidos:** Se "Não se aplica" substituiu um valor, remover a linha
3. **Campos adicionados:** Se um valor substituiu "Não se aplica", adicionar a linha

> A linguagem do Backend é um campo fundamental — não muda numa modificação, então os arquivos
> complementares do Backend (`api.md`, `services.md`, `tests.md`) nunca são re-processados aqui.
> (Eles só são tratados ao **adicionar** um Backend novo — ver 9.2.)

### 9.4 — Guias inativos

Não modifique guias que não foram selecionados como ativos.

### 9.5 — Validação Pós-Aplicação

Antes da confirmação final, execute o checklist canônico da seção **"Validação
Pós-Aplicação"** de `.claude/skills/sdd.references/file-application.md` (aplique os itens
gerais e os marcados `[evolve]`).

---

## Fase 10 — Confirmação Final

Após aplicar todas as mudanças, exiba um resumo compacto:

```
## Evolução concluída

**Arquivos modificados:** <quantidade>
- CLAUDE.md
- .specs/architecture/backend/spec.md
- ...

**Camadas adicionadas:** <lista ou "Nenhuma">
**Camadas modificadas:** <lista ou "Nenhuma">
**Camadas mantidas:** <lista das ativas não alteradas>

**Resumo das alterações:**
- <alteração 1>
- <alteração 2>
- ...
```

---

## Tratamento de Cenários Especiais

### Nenhuma camada não-ativa para adicionar

Se o usuário escolher "Adicionar camada(s)" mas todas as 4 camadas já estiverem ativas:

> Todas as camadas de arquitetura já estão ativas neste projeto. Não há camadas para adicionar.
> Use "Modificar camada(s) existente(s)" se precisar alterar alguma configuração.

### Nenhuma camada ativa para modificar

Se o usuário escolher "Modificar camada(s) existente(s)" mas nenhuma camada estiver ativa:

> Nenhuma camada de arquitetura está ativa neste projeto. Execute o `sdd.setup` primeiro
> para configurar as camadas iniciais.

### Projeto sem `sdd.setup` prévio

Se o `CLAUDE.md` ainda contiver placeholders `[ex: ...]`, recuse-se a continuar com a
mensagem do gate de validação da **Fase 0 → item 4 (Validação inicial)**.

### Usuário mantém todos os campos

Se o usuário escolher "Manter: ..." em todos os campos de todas as camadas selecionadas
para modificação, aplique o caso **"Todos os campos mantidos"** de
`.claude/skills/sdd.references/modification-pattern.md` (encerre sem modificar arquivos,
com a mensagem canônica de lá).

### Tentativa de trocar campo fundamental

Se o usuário pedir para mudar Framework do Frontend; Linguagem, Runtime ou Framework do Backend;
ou o Banco principal, **recuse** com a saída de emergência da seção "Campos Fundamentais (não
modificáveis)" do `.claude/skills/sdd.references/modification-pattern.md`. Esses campos são a
fundação do projeto — trocá-los é uma reescrita, fora do escopo do `sdd.evolve`.

### Remoção de camada

O `sdd.evolve` **não suporta remoção de camadas**. Se o usuário pedir para remover:

> A remoção de camadas não é suportada pelo `sdd.evolve`. Se precisar desativar uma camada,
> edite manualmente o `CLAUDE.md` e os arquivos em `.specs/architecture/`.
>
> Para desativar:
> 1. No `CLAUDE.md`, troque `[x]` por `[ ]` na linha da camada
> 2. Remova os comandos relacionados da tabela de Comandos do Projeto
> 3. Atualize a Stack principal removendo as tecnologias da camada
