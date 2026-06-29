---
name: sdd.adopt
description: Adota SDD num projeto brownfield (já inicializado, com código real) detectando automaticamente a stack a partir dos arquivos do projeto e preenchendo os placeholders [ex: ...] do CLAUDE.md e dos guias de arquitetura. Use quando o projeto já tem código mas ainda não tem a estrutura SDD configurada — independente da stack, tecnologia ou bibliotecas, mesmo fora das opções pré-configuradas.
disable-model-invocation: true
model: opus
effort: high
---

## O que esse skill faz

Diferente do `sdd.setup` (que parte de um repositório vazio e **pergunta** tudo) e do `sdd.evolve`
(que **modifica** um projeto já configurado), o `sdd.adopt` é **detection-first**: lê o código
existente, **detecta a stack real** e preenche as specs automaticamente, pedindo confirmação só
onde a detecção for ambígua ou faltar evidência.

É **stack-agnóstico**: funciona com qualquer tecnologia, mesmo as que **não existem** no catálogo
de opções pré-configuradas (`question-flows.md`) — Python, Go, Java, Ruby, Rust, .NET, Vue,
Angular, etc. Para isso, valores fora do catálogo são registrados como **texto livre**.

Ao final:
- `CLAUDE.md` e os `spec.md` das camadas detectadas ficam preenchidos com a stack real
- Os comandos vêm dos **scripts reais** do projeto
- Os guias complementares do Backend ganham **exemplos extraídos do próprio código do projeto**

> **Fonte canônica de detecção:** `.claude/skills/sdd.references/stack-detection.md`. Leia-a antes
> da Fase 1 — ela define os sinais de cada ecossistema, o fallback para stacks desconhecidas, o
> modelo de evidência/confiança e a seleção de exemplos para os guias do Backend.

---

## Fase 0 — Gate

Antes de qualquer detecção, valide o estado do repositório:

### 0.1 — Já configurado?

Leia o `CLAUDE.md` **e** cada `spec.md` de guia marcado com `[x]`. Se **nenhum** deles contiver
placeholders `[ex: ...]` nas seções `[PROJETO]`, o projeto já está configurado. Recuse:

> Este projeto já está configurado para SDD. Para alterar a configuração, use `sdd.evolve`.
> O `sdd.adopt` serve para projetos com código real mas **sem** a estrutura SDD ainda preenchida.

### 0.2 — É brownfield?

Verifique se existe **código/manifesto real** além dos próprios templates (procure manifestos como
`package.json`, `composer.json`, `pyproject.toml`, `go.mod`, `pom.xml`, `Gemfile`, `Cargo.toml`,
`*.csproj`, e diretórios de código-fonte). Se o repositório só tem os arquivos de template (sem
manifesto nem código-fonte), sugira o caminho greenfield:

> Não encontrei código de aplicação neste repositório — apenas os templates SDD. Para um projeto
> **novo**, use `sdd.setup` (questionário guiado). O `sdd.adopt` é para projetos que **já têm
> código**.

Se houver placeholders `[ex: ...]` **e** código real, prossiga para a Fase 1.

---

## Fase 1 — Varredura e detecção

> Siga `.claude/skills/sdd.references/stack-detection.md`.

1. **Identifique o(s) manifesto(s) raiz** → ecossistema, linguagem, runtime (tabela "Manifesto raiz").
   Em monorepo, trate cada manifesto como uma origem distinta e registre o caminho na evidência.
2. **Determine as camadas presentes** (tabela "Detecção de camadas presentes"). Só ative camadas
   com pelo menos um sinal forte — não invente camadas.
3. **Colete sinais por camada** (tabelas de Frontend, Backend, Database, DevOps), usando Glob, Grep
   e Read (operações **somente leitura**).

Não pergunte nada ao usuário ainda — esta fase é puramente de coleta.

---

## Fase 2 — Inferência de campos

Para cada camada presente, infira **cada campo** do `spec.md` correspondente como a tripla
`valor · confiança · evidência` (modelo em `stack-detection.md`):

- Os campos de cada camada são os mesmos que o `sdd.evolve` extrai na sua Fase 0:
  - **Frontend:** Framework, Estilização, Estado global, Estado de servidor, Formulários, Roteamento, UI Library
  - **Backend:** Linguagem, Runtime, Framework, ORM / Query builder, Autenticação, Validação
  - **Database:** Banco principal, ORM / Query builder, Migrations, Banco de desenvolvimento, Banco de teste
  - **DevOps:** CI/CD, Containerização, Hospedagem, Monitoramento, Registry de imagens
- Quando o valor detectado **coincide** com uma opção do `question-flows.md`, use o rótulo canônico.
- Quando **não coincide** (stack fora do catálogo), use o **nome real** em texto livre — nunca force
  para uma opção do catálogo.
- **Database × Backend:** se ambos presentes, o ORM do Database é o **mesmo** detectado no Backend —
  não detecte ORM duas vezes.

---

## Fase 3 — Comandos (lidos do projeto)

> Prioridade absoluta: comandos **reais** do projeto. Tabelas de `command-derivation.md` são apenas
> fallback (ver "Brownfield" naquele arquivo).

Leia as fontes de comandos reais (seção "Comandos — onde achar os comandos reais" de
`stack-detection.md`): scripts de `package.json`/`composer.json`, `Makefile`, `Taskfile.yml`,
`justfile`, `pyproject.toml`, etc. Mapeie cada script para a ação correspondente (instalar, dev,
test, lint, typecheck, build) e registre a evidência (arquivo). Só recorra às tabelas de
`command-derivation.md` quando não houver nenhum comando declarado.

---

## Fase 4 — Apresentação com evidências + confirmação

Apresente ao usuário a stack detectada, **por camada**, no formato do **"Template do `sdd.adopt`"**
da seção "Formato de Resumo (Confirmação)" de `.claude/skills/sdd.references/file-application.md`
(com confiança + evidência por campo). Liste também os comandos detectados e os exemplos do Backend
que serão substituídos (com a origem).

Peça ao usuário **confirmar ou corrigir** em texto livre. Valores de confiança **alta** podem ser
aceitos como estão; **média/baixa** devem ser destacados para revisão.

### Modo de arquitetura (quando Frontend e Backend coexistem)

Determine o modo por evidência:
- Sinais de Inertia.js, Livewire, Blade, Twig, HTMX, EJS/Handlebars, ou templates server-side
  servidos pelo backend → **Integrado**.
- Frontend com bundler/servidor próprio + Backend expondo API → **Separados**.
- Se a evidência não permitir decidir com confiança, **pergunte** usando o bloco
  **"Modo de Arquitetura"** de `.claude/skills/sdd.references/question-flows.md`.

---

## Fase 5 — Preenchimento de lacunas

Para cada campo **não detectado** ou de **confiança baixa** não resolvido na Fase 4:

- Se a stack da camada é **pré-configurada** (o valor pertenceria ao catálogo): use o bloco
  `AskUserQuestion` da seção correspondente em `question-flows.md` (com "Outra" disponível).
- Se a stack é **fora do catálogo**: pergunte em **texto livre** — os blocos do `question-flows.md`
  são específicos de ecossistema e não se aplicam.

Não invente valores para preencher lacunas — pergunte.

---

## Fase 6 — Confirmação final

Exiba o resumo consolidado (já no formato do "Template do `sdd.adopt`"), agora com as lacunas
preenchidas e correções aplicadas. Em seguida, use a **"Pergunta de confirmação"** canônica da
seção "Formato de Resumo (Confirmação)" de `file-application.md`.

- Se **Reiniciar**: volte à Fase 4 (a detecção das Fases 1–3 é preservada; refaça a revisão).
- Se **Confirmar e aplicar**: prossiga para a Fase 7.

---

## Fase 7 — Aplicação das Mudanças

> Leia `.claude/skills/sdd.references/file-application.md` para as instruções canônicas de edição.

Edite os arquivos um de cada vez:

### 7.1 — CLAUDE.md

Aplique a seção "CLAUDE.md" do arquivo de referência (notas **No `sdd.adopt`**):
1. Remover o bloco de instruções do topo
2. Preencher Identidade do Projeto (Nome derivado do projeto e confirmado, Descrição, Stack principal)
3. Marcar `[x]` nas camadas **detectadas como presentes**
4. Substituir o placeholder da tabela de comandos pelos comandos **reais** (Fase 3)
5. Adicionar links das Referências Rápidas (se houver — ex: README, docs do projeto)

### 7.2 — Specs das camadas presentes (`.specs/architecture/*/spec.md`)

Aplique a seção "Specs de Arquitetura — spec.md" (procedimento padrão + nota **No `sdd.adopt`**):
1. Remover o bloco de instruções do topo
2. Preencher `## Stack [PROJETO]` com os valores detectados (valores livres quando fora do catálogo)
3. Remover linhas de campos "Não se aplica"

### 7.3 — Backend: exemplos a partir do código real

Aplique a seção **"No `sdd.adopt` — Backend brownfield: exemplos a partir do código real"** de
`file-application.md` (mecânica de seleção/normalização em `stack-detection.md`): para `api.md`,
`services.md`, `tests.md` e as partes de linguagem de `spec.md`, **substitua** os exemplos curados
por trechos do código real (anotando a origem) ou remova com nota quando não houver código
representativo. **Princípios e prosa permanecem intactos.**

### 7.4 — Guias inativos

Não modifique guias de camadas **não detectadas**.

---

## Fase 8 — Validação pós-aplicação + relatório

### 8.1 — Validação

Execute o checklist canônico da seção **"Validação Pós-Aplicação"** de `file-application.md`
(itens gerais + os marcados `[adopt]`).

### 8.2 — Relatório final

Exiba um resumo compacto:

```
## Adoção concluída

**Arquivos modificados:** <quantidade>
- CLAUDE.md
- .specs/architecture/<camada>/spec.md
- ...

**Camadas detectadas:** <lista>
**Detectado automaticamente:** <nº de campos com confiança alta>
**Confirmado/corrigido pelo usuário:** <nº de campos>
**Preenchido por pergunta (lacunas):** <nº de campos>

**Exemplos do Backend substituídos:**
- services.md ← <arquivo de origem>
- api.md ← <arquivo de origem>
- <tipo> → removido (sem código representativo)

**Comandos (origem):**
- npm run dev ← package.json
- ...

**Pendências:** <placeholders restantes e o motivo, ou "Nenhuma">
```

---

## Tratamento de Cenários Especiais

### Projeto já configurado
Sem placeholders `[ex: ...]` em CLAUDE.md + specs ativos → recuse apontando `sdd.evolve`
(mensagem do gate **Fase 0 → 0.1**).

### Repositório sem código de aplicação
Só com os templates SDD → sugira `sdd.setup` (mensagem do gate **Fase 0 → 0.2**).

### Stack totalmente fora do catálogo
Nenhum sinal coincide com o `question-flows.md` → use **inteiramente** o fallback de texto livre
de `stack-detection.md`. Detecte via manifesto/dependências, registre valores reais e confirme com
o usuário. Isto é esperado e suportado — não é erro.

### Monorepo / múltiplas linguagens no Backend
Se houver mais de um serviço backend com linguagens distintas, registre na evidência e **pergunte ao
usuário** qual stack a spec de Backend deve refletir (ou se o projeto deve ser tratado por serviço).
Não misture linguagens numa única spec.

### Detecção conflitante
Se dois sinais apontarem valores diferentes para o mesmo campo (ex: dois ORMs nas deps), apresente
ambos com as evidências na Fase 4 e deixe o usuário decidir — não escolha silenciosamente.
