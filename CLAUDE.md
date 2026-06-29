# CLAUDE.md

> Arquivo lido pelo Claude Code no início de cada sessão.
> Serve como orientador: onde as coisas ficam, como trabalhar, o que nunca violar.
> Customize as seções marcadas com [PROJETO] antes de iniciar.
> Remover esse bloco ao iniciar um novo projeto. O arquivo deve começar a partir da seção `Identidade do Projeto`

---

## Identidade do Projeto [PROJETO]

- **Nome:** [ex: strava-dashboard]
- **Descrição curta:** [ex: Dashboard web para visualização de atividades do Strava]
- **Stack principal:** [ex: React 18 + TypeScript + Tailwind]

---

## Princípios do Projeto

> Princípios que regem o desenvolvimento. Valem para qualquer domínio, linguagem ou stack.
> Regras específicas de domínio ficam nos guias de arquitetura.
> Itens marcados com `[PADRÃO]` são boas práticas gerais — mantenha salvo razão específica. Itens sem a marca são absolutos.

### Artigo 1 — Processo SDD [PADRÃO]

O fluxo de desenvolvimento segue a sequência definida na seção "Fluxo de Trabalho Obrigatório" abaixo.

- Nenhuma implementação começa sem uma spec aprovada — `spec.md` ou outro `*.md` que siga a "Estrutura de Artefatos SDD" —, derivada de frameworks e/ou toolkits (`OpenSpec`, `Spec Kit`, `Superpowers`, `SpecStory`, etc.) ou escrita manualmente
  - Uma spec está aprovada quando o responsável pelo projeto confirma explicitamente que ela representa o escopo desejado — seja por mensagem no chat, aprovação de PR ou checklist preenchido
- Specs escritas manualmente seguem a mesma estrutura que as geradas por ferramentas de SDD
- O plano de implementação deve justificar decisões não-óbvias com raciocínio explícito
- Mudanças de requisito durante a implementação atualizam a spec — nunca o contrário
- Uma spec cobre uma feature completa do ponto de vista do usuário — algo que pode ser implementado, testado e entregue em um PR ou sequência curta de PRs. Features independentes têm specs independentes; variações de uma mesma feature pertencem à mesma spec

### Artigo 2 — Qualidade de Código [PADRÃO]

- Proibido deixar código morto (funções, variáveis, imports não utilizados)
- Proibido comentários do tipo `// TODO` sem issue associada
- Nomes de variáveis e funções devem revelar intenção — sem abreviações obscuras
- Siga também as recomendações de `Qualidade de Código` dos guias de arquitetura ativos em `.specs/architecture/` (frontend, backend, database, devops)

### Artigo 3 — Tratamento de Erros [PADRÃO]

- Todo erro é tratado — proibido `try/catch` vazio
- Erros são logados com contexto suficiente para reprodução (o quê falhou, onde, com quais dados)
- Proibido logar dados sensíveis (senhas, tokens, CPF, cartões, dados pessoais identificáveis)
- Proibido expor mensagens de erro técnicas diretamente ao usuário final
- Estados de falha são tão planejados quanto os estados de sucesso

### Artigo 4 — Dependências [PADRÃO]

- Nenhuma dependência nova entra sem avaliação de: manutenção ativa, tamanho, alternativa nativa
- Dependências de desenvolvimento não entram em produção
- Prefira versões fixadas em produção; ranges abertos (`^`, `~`) são aceitáveis desde que com lockfile commitado

### Artigo 5 — Testes [PADRÃO]

- Toda lógica de negócio nova entra com testes — proibido merge sem cobertura da feature
- Testes são independentes — proibido depender de ordem de execução, banco compartilhado entre testes ou estado global persistido
- Testes validam comportamento observável — proibido testar detalhes de implementação (estado interno, chamadas internas)
- Obrigatório: cobertura mínima de 80% de branches nas camadas de lógica — componentes, hooks e utils no frontend; services no backend
- Detalhes de framework, fixtures e mocking ficam nos guias `*/tests.md` da camada

---

## Convenção de Severidade

As regras deste projeto seguem uma hierarquia inspirada na RFC 2119:

| Termo | Significado |
|-------|-------------|
| **Proibido** | Violação é erro — não passa em review |
| **Obrigatório** | Deve estar presente — ausência é erro |
| **Sempre** | Regra sem exceção — salvo documentado e aprovado |
| **Prefira / Evite** | Recomendação forte — exceções justificadas no PR |
| **Considere / Pode** | Sugestão — use seu julgamento |

> Regras marcadas com `[PADRÃO]` podem ser alteradas se o projeto tiver razão específica.

---

## Estrutura de Artefatos SDD

Os artefatos do projeto seguem esta hierarquia. Leia-os nesta ordem antes de qualquer implementação:

```
CLAUDE.md                  ← Princípios do projeto + orientação de sessão (este arquivo)
.specs/
  architecture/
    frontend/
      spec.md              ← orientações para arquitetura da camada de frontend (UI, componentes, testes de interface, acessibilidade)
      components.md
      hooks.md
      utils.md
      tests.md
    backend/
      spec.md              ← orientações para arquitetura do backend (APIs, autenticação, segurança, testes de serviço)
      api.md
      services.md
      tests.md
    database/
      spec.md              ← orientações para modelagem, acesso e performance do banco de dados (migrations, queries, convenções)
      modeling.md
      migrations.md
      queries.md
    devops/
      spec.md              ← orientações para processos de DevOps (CI/CD, ambientes, automação, secrets, build/deploy)
      ci.md
      containers.md
      environments.md
      monitoring.md
```

> Esta estrutura é a **referência canônica** do projeto para qualquer framework ou toolkit de SDD
> (OpenSpec, Spec Kit, Superpowers, SpecStory, etc.). Independente da ferramenta utilizada,
> as specs podem ser escritas manualmente ou geradas por qualquer ferramenta — o formato é o mesmo.

### Guias de Arquitetura Ativos [PROJETO]

Marque com `[x]` apenas os guias que se aplicam a este projeto:

- [ ] `.specs/architecture/frontend/spec.md`
- [ ] `.specs/architecture/backend/spec.md`
- [ ] `.specs/architecture/database/spec.md`
- [ ] `.specs/architecture/devops/spec.md`

> Leia os guias ativos antes de qualquer implementação — eles complementam os princípios deste arquivo.

---

## Delegação a Agentes Especializados

Cada camada ativa tem um subagente dedicado em `.claude/agents/`, que segue à risca o `AGENTS.md` da sua pasta em `.specs/architecture/`.

> **Sempre** que uma tarefa envolver uma camada ativa (frontend, backend, database ou devops), é **obrigatório** delegá-la ao agente correspondente. Não implemente código de camada direto no contexto principal — invoque o especialista.

| Camada | Agente | Definição que ele segue |
|--------|--------|-------------------------|
| Frontend | `frontend` | [`.specs/architecture/frontend/AGENTS.md`](.specs/architecture/frontend/AGENTS.md) |
| Backend | `backend` | [`.specs/architecture/backend/AGENTS.md`](.specs/architecture/backend/AGENTS.md) |
| Database | `database` | [`.specs/architecture/database/AGENTS.md`](.specs/architecture/database/AGENTS.md) |
| DevOps | `devops` | [`.specs/architecture/devops/AGENTS.md`](.specs/architecture/devops/AGENTS.md) |

Tarefas que **cruzam camadas** (ex: backend + frontend) são coordenadas pelo agente principal: os subagentes não se comunicam entre si. O principal divide o trabalho, passa o contrato (API, tipos, schema) de um agente para o outro e integra os resultados.

---

## Fluxo de Trabalho Obrigatório

Siga esta sequência. Nunca pule etapas.

```
1. Ler os Princípios do Projeto (acima)
2. Criar ou receber uma spec aprovada
3. Elaborar o plano de implementação baseado na spec
4. Quebrar o plano em tarefas atômicas e ordenadas
5. Implementar tarefa por tarefa
6. Validar contra a spec antes de fechar
```

### Quando parar e esclarecer (antes do passo 3)

Esclareça os requisitos antes de elaborar o plano quando:

- O pedido do usuário está vago ou pode ser interpretado de mais de uma forma
- A spec levanta perguntas que bloqueiam decisões de arquitetura
- Há dependências externas (integrações, regras de negócio, restrições) que não foram mencionadas
- O escopo não está claro — o que está dentro e o que está fora da feature

> Não avance para o plano com ambiguidades que vão forçar suposições. Esclareça primeiro.

**Nunca abra PR sem todos os critérios de done satisfeitos.**

---

## Definition of Done

Uma tarefa está concluída quando:

- Código implementado conforme o plano de implementação
- Testes escritos e passando
- Sem erros de lint ou de tipagem
- Princípios de qualidade de código respeitados (Artigo 2)
- Spec atualizada com as decisões tomadas durante a implementação

Uma feature está concluída quando:

- Todas as tarefas da feature finalizadas
- Comportamento validado contra a spec
- Sem regressões nas features anteriores

> Os itens de lint, tipagem e testes só são verificáveis após preencher a tabela "Comandos do Projeto" (ou rodar a skill `sdd.setup` em projeto novo, ou `sdd.adopt` em projeto já existente).

---

## Comandos do Projeto [PROJETO]

| Ação | Comando | Camada |
|------|---------|--------|
| [ex: preencha com os comandos reais; ou use `sdd.setup` (projeto novo) / `sdd.adopt` (projeto já existente)] | | |

---

## Referências Rápidas

- **Princípios do projeto:** seção "Princípios do Projeto" (acima)
- **Specs ativas:** `.specs/`
- **Guias de arquitetura:** `.specs/architecture/`
- **Agentes especializados:** `.claude/agents/` — um por camada ativa, seguindo o `AGENTS.md` da pasta (ver "Delegação a Agentes Especializados")