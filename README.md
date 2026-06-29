# sdd-templates

Templates base para projetos usando Spec-Driven Development (SDD). São agnósticos de ferramenta: funcionam com qualquer assistente de código (Claude Code, Cursor, etc.) e com qualquer fluxo de SDD, manual ou automatizado. Todos os arquivos deste repositório devem ser usados como templates de auxílio — principalmente o `CLAUDE.md`, que reúne os princípios não-negociáveis e a orientação de sessão.

Os arquivos para definição de arquitetura como frontend, backend, database e devops, devem ser atualizados conforme a necessidade do projeto e suas stacks. A pasta pode ser copiada em sua totalidade e colada no projeto de destino.

## Estrutura

```
sdd-templates/
  CLAUDE.md                              ← princípios não-negociáveis + orientação de sessão
  README.md                              ← este arquivo
  .specs/
    architecture/
      frontend/
        spec.md                          ← stack, componentes, testes de UI, acessibilidade
        components.md                    ← padrões e exemplos de componentes React
        hooks.md                         ← padrões e exemplos de hooks customizados
        utils.md                         ← padrões e exemplos de funções utilitárias
        tests.md                         ← guia de testes frontend (Jest/Vitest, RTL, MSW)
      backend/
        spec.md                          ← API, autenticação, segurança, testes de serviço
        api.md                           ← padrões de controllers, validação e middleware
        services.md                      ← padrões de serviços e regras de negócio
        tests.md                         ← guia de testes backend (unitários e integração)
      database/
        spec.md                          ← modelagem, migrations, acesso a dados, performance
        modeling.md                      ← schema design, tipos, relacionamentos, soft delete
        migrations.md                    ← criação, alterações seguras, data migrations, seeds, rollback
        queries.md                       ← repositórios, CRUD, paginação, transações, N+1, performance
      devops/
        spec.md                          ← CI/CD, ambientes, Docker, branching, secrets
        ci.md                            ← pipelines, stages, caching, deploy, workflows reutilizáveis
        containers.md                    ← Dockerfiles, Compose, multi-stage, segurança de imagem
        environments.md                  ← variáveis, secrets, .env patterns, feature flags
        monitoring.md                    ← logging, métricas, health checks, alerting
  .claude/
    skills/
      sdd.setup/
        SKILL.md                         ← skill de setup guiado (questionário + preenchimento automático)
      sdd.adopt/
        SKILL.md                         ← skill de adoção brownfield (detecta a stack existente + preenche)
      sdd.evolve/
        SKILL.md                         ← skill de evolução (adiciona camadas, modifica stacks existentes)
      sdd.references/
        question-flows.md                ← opções canônicas de todas as perguntas do questionário
        addition-flow.md                 ← sequência de adição de camada (setup e evolve)
        modification-pattern.md          ← padrão "Manter" e cascata de modificação (evolve)
        command-derivation.md            ← tabelas de derivação de comandos por stack
        stack-detection.md               ← detecção de stack a partir do código (adopt)
        file-application.md              ← instruções de edição de CLAUDE.md, specs e arquivos complementares
```

## Marcadores

- `[PROJETO]` → marcador em cabeçalhos de seção — indica que todo o conteúdo abaixo é configurável por projeto e pode ser modificado pelas skills `sdd.setup`, `sdd.adopt` e `sdd.evolve`
- `[PADRÃO]` → boas práticas gerais — mantenha salvo razão específica para mudar

## Skills

Este repositório inclui três skills complementares que cobrem o ciclo de vida da configuração SDD: **`sdd.setup`** (projeto novo / greenfield), **`sdd.adopt`** (projeto já existente / brownfield) e **`sdd.evolve`** (evolução contínua).

### `sdd.setup` — Configuração inicial (greenfield)

Usada ao iniciar um **projeto novo**, sem código ainda. Conduz um questionário guiado que coleta todas as informações e preenche os templates automaticamente.

**Fluxo:**

1. **Identidade do Projeto** — nome, descrição, slug
2. **Guias de Arquitetura Ativos** — quais camadas se aplicam (frontend, backend, database, devops)
3. **Modo de Arquitetura** — Separados ou Integrado (quando Frontend e Backend co-existem)
4. **Stack dos Guias Ativos** — framework, ORM, estilização, autenticação e demais escolhas técnicas por camada, com opções condicionais por ecossistema
5. **Comandos do Projeto** — derivação automática dos comandos de dev, test, lint e build a partir das stacks
6. **Referências Rápidas** — links externos opcionais (docs, Figma, board, etc.)

Ao final, exibe um resumo para confirmação e aplica todas as mudanças no `CLAUDE.md` e nos guias de arquitetura ativos.

### `sdd.adopt` — Adoção em projeto existente (brownfield)

Usada quando o projeto **já tem código real** mas ainda não tem a estrutura SDD preenchida. Em vez de perguntar tudo, é **detection-first**: lê os arquivos do projeto, **detecta a stack** e preenche as specs automaticamente — pedindo confirmação só onde a detecção é ambígua ou falta evidência. É **stack-agnóstica**: funciona com qualquer tecnologia, mesmo fora das opções pré-configuradas (Python, Go, Java, Ruby, Rust, .NET, Vue, Angular, etc.), registrando valores fora do catálogo como texto livre.

**Fluxo:**

1. **Gate** — recusa se já configurado (aponta `sdd.evolve`); sugere `sdd.setup` se não houver código de aplicação
2. **Varredura e detecção** — identifica manifestos, camadas presentes e sinais de stack (`stack-detection.md`)
3. **Inferência de campos** — cada campo das specs como `valor · confiança · evidência`
4. **Comandos** — lidos dos **scripts reais** do projeto (`package.json`, `Makefile`, etc.), não adivinhados
5. **Apresentação com evidências** — mostra o detectado por camada; usuário confirma ou corrige
6. **Preenchimento de lacunas** — pergunta só o que não foi detectado ou tem baixa confiança
7. **Aplicação** — preenche `CLAUDE.md` e specs; substitui os exemplos dos guias de Backend por **trechos do código real** do projeto (selecionar + normalizar, anotando a origem)
8. **Relatório** — o que foi detectado × confirmado × perguntado, exemplos substituídos e pendências

> **Importante:** A substituição de exemplos por código real acontece **no projeto-alvo**. Os exemplos curados deste template (TS/JS + PHP) permanecem intactos para uso com `sdd.setup`.

### `sdd.evolve` — Evolução de projeto existente

Usada **após** o `sdd.setup`, quando o projeto cresce e precisa de novas camadas ou alterações nas escolhas atuais.

**Fluxo:**

1. **Detecção do Estado Atual** — lê o `CLAUDE.md` e as specs ativas para mapear as stacks, comandos e guias já configurados
2. **Tipo de Evolução** — adicionar novas camadas, modificar existentes, ou ambos
3. **Novas Camadas** — mesmo questionário do `sdd.setup`, com opções do zero
4. **Camadas Existentes** — questionário com opção "Manter: `<valor atual>`" em cada campo, preservando o que não mudou
5. **Sincronização Cross-Layer** — verifica consistência entre camadas (ex: ORM do Backend com Database, modo Integrado)
6. **Comandos** — re-deriva comandos incrementalmente (mantém os de camadas não alteradas)
7. **Aplicação** — edita apenas o que foi adicionado ou modificado, com indicadores visuais (`⚡` alterado, `✨` novo, `mantido`)

> **Importante:** O `sdd.evolve` não suporta remoção de camadas. Para remover, edite os arquivos manualmente no `CLAUDE.md` (desmarcar `[x]`) e nas specs correspondentes.

## Arquivos de Referência (`sdd.references/`)

Extraídos para evitar duplicação entre as skills. Contêm o conteúdo canônico que elas consomem:

| Arquivo | Conteúdo | Consumido por |
|---------|----------|---------------|
| `question-flows.md` | Blocos `AskUserQuestion` com todas as opções de stack, organizados sob seções `## ... [PROJETO]` que indicam campos que aceitam preservação ("Manter:") no `sdd.evolve` | setup, evolve, adopt (lacunas) |
| `addition-flow.md` | Sequência canônica de adição de uma camada do zero (ordem, passos, combinação de campos) | setup, evolve |
| `modification-pattern.md` | Padrão "Manter", campos fundamentais e cascata de modificação | evolve |
| `command-derivation.md` | Tabelas de mapeamento stack → comandos + resolução de conflito Docker (fallback do `adopt`) | setup, evolve, adopt |
| `stack-detection.md` | Detecção da stack a partir do código (sinais por ecossistema, fallback para stacks fora do catálogo, modelo de evidência/confiança, seleção de exemplos do Backend) | adopt |
| `file-application.md` | Instruções canônicas de edição do `CLAUDE.md`, `spec.md` e arquivos complementares do Backend | setup, evolve, adopt |

**Ganho:** Se uma opção de pergunta muda (ex: adicionar "Next.js 15"), edita-se apenas `question-flows.md`. Todas as skills refletem a mudança automaticamente.

## Como usar em um novo projeto

**1. Copie os arquivos para o projeto:**
```bash
cp CLAUDE.md /seu-projeto/
cp -r .specs /seu-projeto/
cp -r .claude /seu-projeto/
```

**2. Rode a skill de configuração — depende do tipo de projeto:**

- **Projeto novo (greenfield):** rode `sdd.setup`. A skill conduz um questionário guiado que coleta todas as informações e preenche os templates automaticamente.
- **Projeto já existente (brownfield):** rode `sdd.adopt`. A skill **detecta** a stack a partir do código, preenche as specs com os valores reais (inclusive tecnologias fora do catálogo) e substitui os exemplos dos guias de Backend por trechos do próprio código do projeto.

Em ambos os casos, ao final a skill exibe um resumo para confirmação e aplica as mudanças no `CLAUDE.md` e nos guias de arquitetura ativos.

**3. Quando o projeto evoluir, rode `sdd.evolve`:**

Se o projeto ganhar novas camadas (ex: começou só com Frontend e depois precisou de Backend + Database) ou precisar alterar stacks existentes (ex: trocar de React Router para TanStack Router), use `sdd.evolve`. A skill preserva tudo que não for explicitamente alterado.

## Como compor por tipo de projeto

| Tipo de projeto | Guias ativos |
|---|---|
| Frontend SPA | `frontend/spec.md` |
| API REST | `backend/spec.md` + `database/spec.md` |
| Fullstack | `frontend/spec.md` + `backend/spec.md` + `database/spec.md` |
| Fullstack com deploy | todos os quatro |
| Microsserviço | `backend/spec.md` + `database/spec.md` + `devops/spec.md` |

> Cada `spec.md` puxa automaticamente seus guias complementares (`components.md`, `api.md`, etc.) — não é necessário ativá-los separadamente.

## Como manter este repositório atualizado

Sempre que aprender algo novo em um projeto:

- **Melhorou uma regra?** Traga para o guia correspondente
- **Criou uma nova convenção?** Adicione na seção certa
- **Encontrou um anti-pattern?** Documente como proibição explícita
- **Mudou de stack?** Atualize a seção `[PROJETO]` correspondente — ou crie uma variante

O objetivo é que este repositório reflita suas opiniões atuais sobre como construir software.
