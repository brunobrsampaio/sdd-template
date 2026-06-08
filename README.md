# sdd-templates

Templates base para projetos usando Spec-Driven Development (SDD). São agnósticos de ferramenta: funcionam com qualquer assistente de código (Claude Code, Cursor, etc.) e com qualquer fluxo de SDD, manual ou automatizado. Todos os arquivos deste repositório devem ser usados como templates de auxílio — principalmente o `CLAUDE.md`, que reúne os princípios não-negociáveis e a orientação de sessão.

Os arquivos para definição de arquitetura como frontend, backend, database e devops, devem ser atualizados conforme a necessidade do projeto e suas stacks. A pasta pode ser copiada em sua totalidade e colada no projeto de destino.

## Estrutura

```
sdd-templates/
  CLAUDE.md                              ← princípios não-negociáveis + orientação de sessão
  README.md                              ← este arquivo
  .specs/
    changelog.md                         ← histórico de decisões (princípios, guias, convenções)
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
      devops/
        spec.md                          ← CI/CD, ambientes, Docker, branching, secrets
  .claude/
    skills/
      sdd.setup/
        SKILL.md                         ← skill de setup guiado (questionário + preenchimento automático)
```

## Marcadores

- `[PROJETO]` → blocos preenchidos automaticamente pela skill `sdd.setup` — específicos por projeto
- `[PADRÃO]` → boas práticas gerais — mantenha salvo razão específica para mudar

## Como usar em um novo projeto

**1. Copie os arquivos para o projeto:**
```bash
cp CLAUDE.md /seu-projeto/
cp -r .specs /seu-projeto/
cp -r .claude /seu-projeto/
```

**2. Rode a skill `sdd.setup`:**

A skill conduz um questionário guiado que coleta todas as informações do projeto e preenche os templates automaticamente. O processo passa por 5 fases:

1. **Identidade** — nome e descrição do projeto
2. **Camadas ativas** — quais guias de arquitetura se aplicam (frontend, backend, database, devops)
3. **Stack** — framework, ORM, estilização, autenticação e demais escolhas técnicas por camada
4. **Comandos** — derivação automática dos comandos de dev, test, lint e build
5. **Referências** — links externos opcionais (docs, Figma, board, etc.)

Ao final, a skill exibe um resumo para confirmação e aplica todas as mudanças no `CLAUDE.md` e nos guias de arquitetura ativos.

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
- **Mudou de stack?** Atualize o `[PROJETO]` do guia — ou crie uma variante

O objetivo é que este repositório reflita suas opiniões atuais sobre como construir software.
Use o [`.specs/changelog.md`](.specs/changelog.md) para decisões que mudaram de direção.
