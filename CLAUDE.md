# CLAUDE.md

> Arquivo lido pelo Claude Code no início de cada sessão.
> Serve como orientador: onde as coisas ficam, como trabalhar, o que nunca violar.
> Customize as seções marcadas com [PROJETO] antes de iniciar.
> Remover esse bloco ao iniciar um novo projeto. O arquivo deve começar a partir da seção `Identidade do Projeto`

---

## Identidade do Projeto

- **Nome:** [PROJETO - ex: strava-dashboard]
- **Descrição curta:** [PROJETO - ex: Dashboard web para visualização de atividades do Strava]
- **Stack principal:** [PROJETO - ex: React 18 + TypeScript + Tailwind]

---

## Princípios Não-Negociáveis

> Princípios universais não-negociáveis. Valem para qualquer domínio, linguagem ou stack.
> Regras específicas de domínio ficam nos guias de arquitetura.
> Seções marcadas com [PADRÃO] são boas práticas gerais — mantenha salvo razão específica.

### Artigo 1 — Processo SDD [PADRÃO]

O fluxo de desenvolvimento segue a sequência definida na seção "Fluxo de Trabalho Obrigatório" abaixo.

- Nenhuma implementação começa sem `spec.md` ou arquivo `*.md`, derivada de frameworks e/ou toolkits (`OpenSpec`, `Spec Kit`, `Superpowers`, `SpecStory`, etc.)
- Specs escritas manualmente seguem a mesma estrutura que as geradas por ferramentas de SDD
- O plano de implementação deve justificar decisões não-óbvias com raciocínio explícito
- Mudanças de requisito durante a implementação atualizam a spec — nunca o contrário
- O contexto de sessão é atualizado ao final de cada sessão com o que foi decidido e por quê

### Artigo 2 — Qualidade de Código [PADRÃO]

- Proibido deixar código morto (funções, variáveis, imports não utilizados)
- Proibido comentários do tipo `// TODO` sem issue associada
- Nomes de variáveis e funções devem revelar intenção — sem abreviações obscuras
- A qualidade de código deve seguir as recomendações específicas de cada arquitetura (frontend, backend, devops e database) descritas nos respectivos blocos de `Qualidade de Código` dos `Guias de Arquitetura ativos` — essas recomendações estão baseadas nos arquivos em `.specs/architecture/` deste projeto.

### Artigo 3 — Tratamento de Erros [PADRÃO]

- Todo erro é tratado — proibido `try/catch` vazio
- Erros são logados com contexto suficiente para reprodução (o quê falhou, onde, com quais dados)
- Proibido expor mensagens de erro técnicas diretamente ao usuário final
- Estados de falha são tão planejados quanto os estados de sucesso

### Artigo 4 — Dependências [PADRÃO]

- Nenhuma dependência nova entra sem avaliação de: manutenção ativa, tamanho, alternativa nativa
- Dependências de desenvolvimento não entram em produção
- Versões são fixadas — proibido ranges abertos em produção (`^`, `~` com cautela)

### Artigo 5 — Testes [PADRÃO]

- Toda lógica de negócio nova entra com testes — proibido merge sem cobertura da feature
- Testes são independentes — proibido depender de ordem de execução, banco compartilhado entre testes ou estado global persistido
- Testes validam comportamento observável — proibido testar detalhes de implementação (estado interno, chamadas internas)
- Cobertura mínima de 80% de branches na camada de lógica das camadas ativas (componentes/hooks/utils no frontend, services no backend)
- Detalhes de framework, fixtures e mocking ficam nos guias `*/tests.md` da camada

---

## Estrutura de Artefatos SDD

Os artefatos do projeto seguem esta hierarquia. Leia-os nesta ordem antes de qualquer implementação:

```
CLAUDE.md                  ← Princípios não-negociáveis + orientação de sessão (este arquivo)
.specs/
  changelog.md             ← histórico de decisões que mudam princípios ou guias
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

## Fluxo de Trabalho Obrigatório

Siga esta sequência. Nunca pule etapas.

```
1. Ler os Princípios Não-Negociáveis (acima)
2. Criar ou receber uma spec aprovada
3. Elaborar o plano de implementação baseado na spec
4. Quebrar o plano em tarefas atômicas e ordenadas
5. Implementar tarefa por tarefa
6. Validar contra a spec antes de fechar
```

Esclareça os requisitos antes de avançar para o próximo passo quando:
- O pedido do usuário está vago ou pode ser interpretado de mais de uma forma
- A spec levanta perguntas que bloqueiam decisões de arquitetura
- Há dependências externas (integrações, regras de negócio, restrições) que não foram mencionadas
- O escopo não está claro — o que está dentro e o que está fora da feature

> Não avance para o plano com ambiguidades que vão forçar suposições. Esclareça primeiro.

**Nunca implemente sem spec aprovada.**
**Nunca abra PR sem todos os critérios de done satisfeitos.**

---

## Definition of Done

Uma tarefa está concluída quando:

- Código implementado conforme o plano de implementação
- Testes escritos e passando
- Sem erros de lint ou de tipagem
- Princípios de qualidade de código respeitados (Artigo 2)
- Contexto de sessão atualizado com decisões tomadas

Uma feature está concluída quando:

- Todas as tarefas da feature finalizadas
- Comportamento validado contra a spec
- Sem regressões nas features anteriores

---

## Comandos do Projeto [PROJETO]

| Ação | Comando | Camada |
|------|---------|--------|
| [PROJETO - preencha com os comandos reais ou use a skill `sdd.setup`] | | |

---

## Referências Rápidas

- **Princípios não-negociáveis:** seção "Princípios Não-Negociáveis" (acima)
- **Specs ativas:** `.specs/`
- **Guias de arquitetura:** `.specs/architecture/`
- **Registro de alterações:** [`.specs/changelog.md`](.specs/changelog.md)