# sdd-templates

Templates base para projetos usando Spec-Driven Development (SDD). São agnósticos de ferramenta: funcionam com qualquer assistente de código (Claude Code, Cursor, etc.) e com qualquer fluxo de SDD, manual ou automatizado. Todos os arquivos deste repositório devem ser usados como templates de auxílio — principalmente o `CLAUDE.md`, que reúne os princípios não-negociáveis e a orientação de sessão.

Os arquivos para definição de arquitetura como frontend, backend, database e devops, devem ser atualizados conforme a necessidade do projeto e suas stacks. A pasta pode ser copiada em sua totalidade e colada no projeto de destino e assim atualizados.

## Estrutura

```
sdd-templates/
  CLAUDE.md                              ← princípios não-negociáveis + orientação de sessão
  README.md                              ← este arquivo
  .specs/
    architecture/
      frontend/
        spec.md                          ← stack, componentes, testes de UI, acessibilidade
      backend/
        spec.md                          ← API, autenticação, segurança, testes de serviço
      database/
        spec.md                          ← modelagem, migrations, acesso a dados, performance
      devops/
        spec.md                          ← CI/CD, ambientes, Docker, branching, secrets
```

## Marcadores

- `[PROJETO]` → substitua por valores reais antes de começar — específico por projeto
- `[PADRÃO]` → boas práticas gerais — mantenha salvo razão específica para mudar

## Como usar em um novo projeto

**1. Copie os arquivos para o projeto:**
```bash
cp CLAUDE.md /seu-projeto/
cp -r specs /seu-projeto/
```

**2. Preencha os `[PROJETO]` no `CLAUDE.md`:**
- Nome e descrição do projeto
- Stack principal e runtime
- Comandos reais (`npm run dev`, `yarn test`, etc.)
- Links relevantes (docs, Figma, etc.)

**3. Ative os guias de arquitetura que se aplicam:**

No `CLAUDE.md`, marque com `[x]` apenas os guias do projeto:
```markdown
- [x] `.specs/architecture/frontend/spec.md`
- [ ] `.specs/architecture/backend/spec.md`   ← desative o que não usar
```

**4. Preencha os `[PROJETO]` em cada guia de arquitetura ativo:**
- Stack específica (framework, ORM, etc.)
- Ajuste thresholds de cobertura de testes se necessário

**5. (Opcional) Use o setup guiado:**

Em vez de preencher os `[PROJETO]` à mão, rode a skill `sdd.setup` — ela conduz um questionário e preenche o `CLAUDE.md` e os guias de arquitetura ativos automaticamente.

## Como compor por tipo de projeto

| Tipo de projeto | Guias ativos |
|---|---|
| Frontend SPA | `frontend.md` |
| API REST | `backend.md` + `database.md` |
| Fullstack | `frontend.md` + `backend.md` + `database.md` |
| Fullstack com deploy | todos os quatro |
| Microsserviço | `backend.md` + `database.md` + `devops.md` |

## Como manter este repositório atualizado

Sempre que aprender algo novo em um projeto:

- **Melhorou uma regra?** Traga para o guia correspondente
- **Criou uma nova convenção?** Adicione na seção certa
- **Encontrou um anti-pattern?** Documente como proibição explícita
- **Mudou de stack?** Atualize o `[PROJETO]` do guia — ou crie uma variante

O objetivo é que este repositório reflita suas opiniões atuais sobre como construir software.
Use o "Registro de Alterações" do `CLAUDE.md` para decisões que mudaram de direção.
