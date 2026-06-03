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

- Nenhuma implementação começa sem `spec.md` aprovada
- O `plan.md` deve justificar decisões não-óbvias com raciocínio explícito
- Mudanças de requisito durante a implementação atualizam a `spec.md` — nunca o contrário
- O `agent.md` é atualizado ao final de cada sessão com o que foi decidido e por quê
- Specs escritas manualmente seguem a mesma estrutura que as geradas por ferramentas de SDD

### Artigo 2 — Qualidade de Código [PADRÃO]

- Proibido deixar código morto (funções, variáveis, imports não utilizados)
- Proibido `console.log` de debug no código final
- Proibido comentários do tipo `// TODO` sem issue associada
- Toda função pública tem tipagem explícita de entrada e saída
- Nomes de variáveis e funções devem revelar intenção — sem abreviações obscuras

### Artigo 3 — Modularidade [PADRÃO]

- Cada feature começa isolada com fronteiras claras
- O único ponto de entrada público de um módulo é seu `index` (`index.ts`, `__init__.py`, etc.)
- Proibido importar arquivos internos de outro módulo diretamente
- Dependências entre módulos são expressas via interfaces e tipos compartilhados

### Artigo 4 — Tratamento de Erros [PADRÃO]

- Todo erro é tratado — proibido `try/catch` vazio
- Erros são logados com contexto suficiente para reprodução (o quê falhou, onde, com quais dados)
- Proibido expor mensagens de erro técnicas diretamente ao usuário final
- Estados de falha são tão planejados quanto os estados de sucesso

### Artigo 5 — Dependências [PADRÃO → ajuste por projeto]

- Nenhuma dependência nova entra sem avaliação de: manutenção ativa, tamanho, alternativa nativa
- Dependências de desenvolvimento não entram em produção
- Versões são fixadas — proibido ranges abertos em produção (`^`, `~` com cautela)

---

## Estrutura de Artefatos SDD

Os artefatos do projeto seguem esta hierarquia. Leia-os nesta ordem antes de qualquer implementação:

```
CLAUDE.md                  ← Princípios não-negociáveis + orientação de sessão (este arquivo)
specs/
  architecture/
    frontend.md            ← orientações para arquitetura da camada de frontend (UI, componentes, testes de interface, acessibilidade)
    backend.md             ← orientações para arquitetura do backend (APIs, autenticação, segurança, testes de serviço)
    database.md            ← orientações para modelagem, acesso e performance do banco de dados (migrations, queries, convenções)
    devops.md              ← orientações para processos de DevOps (CI/CD, ambientes, automação, secrets, build/deploy)
```

> Specs podem ser escritas manualmente ou geradas por qualquer ferramenta de SDD.
> A estrutura é a mesma nos dois casos.

### Guias de Arquitetura Ativos [PROJETO]

Marque com `[x]` apenas os guias que se aplicam a este projeto:

- [ ] `specs/architecture/frontend.md`
- [ ] `specs/architecture/backend.md`
- [ ] `specs/architecture/database.md`
- [ ] `specs/architecture/devops.md`

> Leia os guias ativos antes de qualquer implementação — eles complementam os princípios deste arquivo.

---

## Fluxo de Trabalho Obrigatório

Siga esta sequência. Nunca pule etapas.

```
1. Ler os Princípios Não-Negociáveis (acima)
2. Criar ou receber spec.md aprovada
3. Gerar plan.md baseado na spec
4. Quebrar plan em tasks.md atômicas
5. Implementar task por task
6. Validar contra spec antes de fechar
```

Esclareça os requisitos antes de avançar para o próximo passo quando:
- O pedido do usuário está vago ou pode ser interpretado de mais de uma forma
- A spec levanta perguntas que bloqueiam decisões de arquitetura no `plan.md`
- Há dependências externas (integrações, regras de negócio, restrições) que não foram mencionadas
- O escopo não está claro — o que está dentro e o que está fora da feature

> Não avance para `plan.md` com ambiguidades que vão forçar suposições. Esclareça primeiro.

**Nunca implemente sem spec.md aprovada.**
**Nunca abra PR sem todos os critérios de done satisfeitos.**

---

## Definition of Done [PROJETO]

Uma task está concluída quando:

- [ ] Código implementado conforme o `plan.md`
- [ ] Testes escritos e passando (`[PROJETO - ex: npm test]`)
- [ ] Sem erros de lint (`[PROJETO - ex: npm run lint]`)
- [ ] Sem TypeScript errors (`[PROJETO - ex: npm run typecheck]`)
- [ ] Sem `console.log` de debug no código final
- [ ] `agent.md` atualizado com decisões tomadas durante a implementação

Uma feature está concluída quando:

- [ ] Todas as tasks da feature finalizadas
- [ ] Comportamento validado contra `spec.md`
- [ ] Sem regressões nas features anteriores

---

## Padronização de Código

A fonte da verdade para estilo e qualidade de código é a configuração do projeto. Não invente ou assuma regras — leia os arquivos de configuração presentes:

- **ESLint** (`eslint.config.*`, `.eslintrc.*`) — regras de qualidade, padrões de código e convenções
- **Prettier** (`.prettierrc`, `prettier.config.*`) — formatação automática (indentação, aspas, ponto-e-vírgula, etc.)

O projeto pode usar um, outro, ou ambos:

| Configuração | Responsabilidade |
|---|---|
| Só ESLint | Qualidade e estilo via lint rules |
| Só Prettier | Formatação automática |
| ESLint + Prettier | ESLint para qualidade, Prettier para formatação — sem sobreposição de regras |

> Toda regra que conflite com ESLint ou Prettier está errada — a configuração do projeto vence.
> Se uma convenção desta seção contradiz o config do projeto, siga o config.

---

## Convenções de Código

> Ajuste conforme o projeto. Seja específico — regras vagas não ajudam.

### Nomenclatura
- Componentes: PascalCase (`UserProfile.tsx`)
- Hooks: camelCase com prefixo `use` (`useUserProfile.ts`)
- Utilitários: camelCase (`formatDate.ts`)
- Constantes: SCREAMING_SNAKE_CASE (`MAX_RETRIES`)
- Arquivos de teste: mesmo nome com `.test` (`UserProfile.test.tsx`)

### Imports
- Sempre importar de `index.ts` de outros módulos
- Proibido importar arquivos internos de outra feature diretamente

---

## Comandos do Projeto

```bash
# [PROJETO - adicione os comandos reais]
npm install       # instalar dependências
npm run dev       # servidor de desenvolvimento
npm test          # rodar testes
npm run lint      # verificar lint
npm run typecheck # verificar tipos
npm run build     # build de produção
```

---

## Referências Rápidas

- Princípios não-negociáveis: seção "Princípios Não-Negociáveis" (acima)
- Specs ativas: `specs/`
- Guias de arquitetura: `specs/architecture/`

---

## Registro de Alterações

> Documente mudanças significativas nos princípios não-negociáveis.
> Cria um histórico de decisões do projeto ao longo do tempo.

| Data | Artigo | Mudança | Motivo |
|------|--------|---------|--------|
| [data] | [artigo] | [o que mudou] | [por que mudou] |