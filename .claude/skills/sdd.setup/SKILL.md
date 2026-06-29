---
name: sdd.setup
description: Configura um novo projeto preenchendo todos os placeholders `[ex: ...]` nas seções [PROJETO] do CLAUDE.md e dos guias de arquitetura. Use quando o usuário iniciar um novo projeto, copiar os templates ou pedir setup do SDD.
disable-model-invocation: true
model: opus
effort: high
---

## O que esse skill faz

Coleta as informações específicas do projeto através de um questionário guiado e substitui todos os
placeholders `[ex: ...]` nas seções marcadas com `[PROJETO]`:

- `CLAUDE.md`
- `.specs/architecture/*/spec.md` (apenas os guias marcados como ativos)

Antes de começar, leia os arquivos listados acima para identificar quais placeholders `[ex: ...]` ainda
não foram preenchidos. A Fase 0 detecta se o projeto já está configurado, considerando tanto o
`CLAUDE.md` quanto os `spec.md` dos guias ativos.

---

## Fase 0 — Detecção do Estado Atual

Antes de iniciar o questionário, leia o `CLAUDE.md` **e** cada `spec.md` de guia marcado com
`[x]` na seção "Guias de Arquitetura Ativos" para verificar se o projeto já foi configurado:

- Se **nenhum** desses arquivos contiver placeholders `[ex: ...]` nas seções `[PROJETO]` — o
  projeto já foi configurado pelo `sdd.setup`. Recuse-se a continuar:

  > Este projeto já foi configurado pelo `sdd.setup`. Para modificar a configuração,
  > use a skill `sdd.evolve`. Para refazer do zero, restaure manualmente os placeholders
  > `[ex: ...]` e execute `sdd.setup` novamente.

- Se **ainda houver placeholders `[ex: ...]`** em qualquer um deles — inclusive no caso em que o
  `CLAUDE.md` está preenchido mas um `spec.md` ativo ainda tem placeholders — o projeto não está
  totalmente configurado. Continue para a Fase 1.

> **Projeto brownfield?** Se o repositório já tem **código de aplicação real** (manifestos como
> `package.json`, `composer.json`, `pyproject.toml`, `go.mod`, etc., e diretórios de código-fonte)
> além dos templates, prefira a skill `sdd.adopt`: ela **detecta** a stack existente e preenche as
> specs automaticamente, em vez de perguntar tudo do zero. Use o `sdd.setup` para projetos novos
> (sem código ainda).

---

## Fase 1 — Identidade do Projeto

Pergunte em texto (entrada livre, uma pergunta por vez):

1. **Nome do projeto** — nome literal, como o usuário chama o projeto (ex: "Strava Dashboard", "API de Pagamentos", "Portal do Cliente")
2. **Descrição curta** — 1 linha descrevendo o que é o projeto

Após receber o nome literal, gere um slug (lowercase, palavras separadas por hífen, sem caracteres
especiais) e confirme com o usuário antes de prosseguir.
Exemplo: "Strava Dashboard" → `strava-dashboard`.

O slug será usado no campo `Nome` do `CLAUDE.md`.

> A stack principal será derivada automaticamente das respostas da Fase 4.

---

## Fase 2 — Guias de Arquitetura Ativos

Use o bloco `AskUserQuestion` da seção **"Seleção de Camadas"** de
`.claude/skills/sdd.references/question-flows.md`.

---

## Fase 3 — Modo de Arquitetura

> Esta fase só se aplica quando **Frontend** e **Backend** foram ambos selecionados na Fase 2.
> Se apenas um deles foi selecionado, pule diretamente para a Fase 4.

Use o bloco `AskUserQuestion` da seção **"Modo de Arquitetura"** de
`.claude/skills/sdd.references/question-flows.md`.

- Se **Separados**: prossiga normalmente para a Fase 4 — Frontend e Backend são tratados de forma independente.
- Se **Integrado**: na Fase 4, execute a seção **Backend** primeiro e, ao terminar, use a seção **Frontend Integrado**. Pule a seção Frontend padrão.

---

## Fase 4 — Stack dos Guias Ativos

> Leia `.claude/skills/sdd.references/addition-flow.md` — fonte canônica da sequência de
> adição de cada camada (ordem de execução, passos e quando combinar campos numa só chamada).
> As opções de cada campo estão em `.claude/skills/sdd.references/question-flows.md`.

Para cada guia selecionado na Fase 2, execute o fluxo da camada correspondente em
`addition-flow.md`. O usuário pode escolher uma das opções apresentadas ou "Outra" para um
valor personalizado.

**Particularidades do `sdd.setup`** (todas as camadas partem do zero):

- **Modo Separados** (Fase 3): configure o Frontend pela seção **Frontend** de `addition-flow.md`.
- **Modo Integrado** (Fase 3): configure o **Backend** primeiro e, em seguida, use a seção
  **Frontend Integrado** de `addition-flow.md`. Pule a seção Frontend padrão.

> A ordem de execução entre camadas (Backend → Database → Frontend → DevOps) e as regras
> cross-layer (ex: Database herdando o ORM do Backend) já estão descritas em `addition-flow.md`.

---

## Fase 5 — Comandos do Projeto

> Leia `.claude/skills/sdd.references/command-derivation.md` para as tabelas canônicas
> de derivação de comandos.

Com base nas camadas ativas e escolhas da Fase 4, derive os comandos usando as tabelas
do arquivo de referência. Apresente as sugestões ao usuário organizadas por camada
e peça confirmação ou correção em texto livre.

Após confirmar, componha a lista final para o `CLAUDE.md` seguindo o formato canônico da
seção **"Comandos do Projeto"** de `.claude/skills/sdd.references/file-application.md`
(uma linha por comando, com coluna "Camada"; ações que não se aplicam são omitidas).

---

## Fase 6 — Referências Rápidas

Pergunte em texto (entrada livre): há links adicionais para as Referências Rápidas?
Exemplos: documentação externa, URL da API, design system, Figma, board do projeto.

Não use `AskUserQuestion` nesta fase — aceite qualquer formato que o usuário queira informar.
Se não houver links, pule esta fase.

---

## Confirmação das Respostas

Antes de aplicar qualquer mudança, exiba um resumo de tudo que foi coletado nas fases
anteriores usando o **"Template do `sdd.setup`"** da seção "Formato de Resumo (Confirmação)"
de `.claude/skills/sdd.references/file-application.md`. Omita seções que não foram
preenchidas (guias inativos, fases puladas).

Em seguida, use a **"Pergunta de confirmação"** canônica da mesma seção do arquivo de referência.

Se o usuário escolher **Reiniciar**, volte ao início da Fase 1 e refaça todo o questionário.
Se confirmar, prossiga para a seção **Aplicação das Mudanças**.

---

## Aplicação das Mudanças

> Leia `.claude/skills/sdd.references/file-application.md` para as instruções canônicas
> de edição de cada arquivo.

Após coletar todas as respostas, edite os arquivos um de cada vez:

### CLAUDE.md

Aplique as instruções da seção "CLAUDE.md" do arquivo de referência:
1. Remover bloco de instruções do topo
2. Preencher Identidade do Projeto (Nome, Descrição curta, Stack principal)
3. Marcar `[x]` nos guias selecionados
4. Substituir placeholder da tabela de comandos pelos comandos reais
5. Adicionar links das Referências Rápidas

### Guias ativos (.specs/architecture/*/spec.md)

Aplique as instruções da seção "Specs de Arquitetura — spec.md" do arquivo de referência:
1. Remover bloco de instruções do topo
2. Preencher `## Stack` com as respostas da Fase 4
3. Remover linhas de campos "não se aplica"

### Arquivos complementares do Backend (.specs/architecture/backend/)

Aplique as instruções da seção "Backend — Arquivos Complementares" do arquivo de referência,
de acordo com a linguagem escolhida na Fase 4:
- Remover exemplos e referências da linguagem não selecionada em `api.md`, `services.md`, `tests.md` e `spec.md`
- Atualizar anti-patterns e tabelas comparativas

### Guias inativos

Não modifique guias que não foram selecionados como ativos na Fase 2.

---

## Validação Pós-Aplicação

Antes da confirmação final, execute o checklist canônico da seção **"Validação
Pós-Aplicação"** de `.claude/skills/sdd.references/file-application.md` (aplique os itens
gerais e os marcados `[setup]`).

---

## Confirmação Final

Após aplicar todas as mudanças, exiba um resumo compacto:

- Arquivos modificados
- Placeholders `[ex: ...]` preenchidos
- Se ficou algum placeholder `[ex: ...]` sem preencher (e o motivo)
