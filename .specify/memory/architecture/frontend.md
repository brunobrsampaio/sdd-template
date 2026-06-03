# Guia de Arquitetura — Frontend

> Regras específicas para projetos com interface web.
> Complementa a constitution — não a substitui.
> Seções marcadas com [PROJETO] devem ser ajustadas por projeto.
> Seções marcadas com [PADRÃO] refletem boas práticas gerais.
> Remover esse bloco ao iniciar um novo projeto. O arquivo deve começar a partir da seção `Stack`

---

## Stack [PROJETO]

- **Framework:** [ex: React 18 com TypeScript — strict mode habilitado]
- **Estilização:** [ex: Tailwind CSS — proibido CSS-in-JS e styled-components]
- **Estado global:** [ex: Zustand — proibido Redux]
- **Estado de servidor:** [ex: TanStack Query — proibido fetch manual em useEffect para dados remotos]
- **Formulários:** [ex: React Hook Form — proibido Formik]
- **Roteamento:** [ex: React Router v6]

---

## Nomenclatura [PADRÃO]

- Componentes: `PascalCase` (`UserProfile.tsx`)
- Hooks: `camelCase` com prefixo `use` (`useUserProfile.ts`)
- Utilitários: `camelCase` (`formatDate.ts`)
- Constantes: `SCREAMING_SNAKE_CASE` (`MAX_RETRIES`)
- Tipos e interfaces: `PascalCase` (`UserProfileProps`)
- Arquivos de teste: mesmo nome com `.test` (`UserProfile.test.tsx`)

---

## Componentes [PADRÃO]

- Componentes com mais de 150 linhas devem ser divididos
- Props sempre tipadas com interface explícita
- Proibido prop drilling além de 2 níveis — use contexto ou estado global
- Componentes são funções — proibido class components
- Efeitos colaterais ficam em hooks, não diretamente nos componentes

---

## Testes [PADRÃO → ajuste thresholds]

- **Framework:** Jest + React Testing Library
- **HTTP mocking:** MSW (Mock Service Worker) — proibido `jest.mock` de `fetch`
- **Abordagem:** TDD — testes escritos antes ou junto com o código
- **Cobertura mínima:** 80% de branches por feature
- **Queries:** sempre semânticas (`getByRole`, `getByLabelText`) — proibido `getByTestId` salvo último recurso
- **O que testar:** comportamento observável pelo usuário, não detalhes de implementação
- **O que não testar:** detalhes internos, estado interno de hooks em isolamento

---

## Acessibilidade [PADRÃO]

- Navegação por teclado funcional em todos os fluxos principais
- Imagens têm `alt` descritivo — `alt=""` apenas para imagens decorativas

---

## Performance [PADRÃO]

- Proibido importar bibliotecas inteiras quando só parte é usada (`import { X } from 'lib'`)
- Lazy loading para rotas e componentes pesados
- Imagens com dimensões explícitas para evitar layout shift
