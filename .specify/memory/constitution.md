# Constitution

> Princípios universais não-negociáveis.
> Valem para qualquer domínio, linguagem ou stack.
> Regras específicas de domínio ficam nos guias de arquitetura.
> Seções marcadas com [PADRÃO] são boas práticas gerais — mantenha salvo razão específica.
> Remover esse bloco ao iniciar um novo projeto. O arquivo deve começar a partir da seção `Artigo 1 — Processo SDD`

---

## Artigo 1 — Processo SDD [PADRÃO]

O fluxo de desenvolvimento segue a sequência definida no `CLAUDE.md`.

- Nenhuma implementação começa sem `spec.md` aprovada
- O `plan.md` deve justificar decisões não-óbvias com raciocínio explícito
- Mudanças de requisito durante a implementação atualizam a `spec.md` — nunca o contrário
- O `agent.md` é atualizado ao final de cada sessão com o que foi decidido e por quê
- Specs escritas manualmente seguem a mesma estrutura que as geradas pelo SpecKit

---

## Artigo 2 — Qualidade de Código [PADRÃO]

- Proibido deixar código morto (funções, variáveis, imports não utilizados)
- Proibido `console.log` de debug no código final
- Proibido comentários do tipo `// TODO` sem issue associada
- Toda função pública tem tipagem explícita de entrada e saída
- Nomes de variáveis e funções devem revelar intenção — sem abreviações obscuras

---

## Artigo 3 — Modularidade [PADRÃO]

- Cada feature começa isolada com fronteiras claras
- O único ponto de entrada público de um módulo é seu `index` (`index.ts`, `__init__.py`, etc.)
- Proibido importar arquivos internos de outro módulo diretamente
- Dependências entre módulos são expressas via interfaces e tipos compartilhados

---

## Artigo 4 — Tratamento de Erros [PADRÃO]

- Todo erro é tratado — proibido `try/catch` vazio
- Erros são logados com contexto suficiente para reprodução (o quê falhou, onde, com quais dados)
- Proibido expor mensagens de erro técnicas diretamente ao usuário final
- Estados de falha são tão planejados quanto os estados de sucesso

---

## Artigo 5 — Dependências [PADRÃO → ajuste por projeto]

- Nenhuma dependência nova entra sem avaliação de: manutenção ativa, tamanho, alternativa nativa
- Dependências de desenvolvimento não entram em produção
- Versões são fixadas — proibido ranges abertos em produção (`^`, `~` com cautela)

---

## Registro de Alterações

> Documente mudanças significativas na constitution.
> Cria um histórico de decisões do projeto ao longo do tempo.

| Data | Artigo | Mudança | Motivo |
|------|--------|---------|--------|
| [data] | [artigo] | [o que mudou] | [por que mudou] |
