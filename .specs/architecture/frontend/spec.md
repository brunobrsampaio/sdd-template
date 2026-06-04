# Guia de Arquitetura — Frontend

> Regras específicas para projetos com interface web.
> Complementa os princípios não-negociáveis do `CLAUDE.md` — não os substitui.
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

## Qualidade e Padronização de Código [PADRÃO]

**A fonte da verdade absoluta para estilo e qualidade de código são os arquivos de configuração do projeto:**

- **ESLint** (`eslint.config.*`, `.eslintrc.*`) — regras de qualidade, convenções e padrões
- **Prettier** (`.prettierrc`, `prettier.config.*`) — formatação (indentação, aspas, ponto-e-vírgula, largura de linha, etc.)

Se esses arquivos existirem no repositório, **nenhuma regra deste guia pode contradizê-los**.
Em caso de conflito, o config do projeto vence — sempre.

As regras abaixo complementam os configs — servem para cobrir o que linters e formatadores não capturam.

### Nomenclatura

- Pastas de componentes: `PascalCase` (`Button/`, `UserProfile/`)
- Hooks: `camelCase` com prefixo `use` (`useUserProfile.ts`)
- Utilitários: `camelCase` (`formatDate.ts`)
- Constantes: `SCREAMING_SNAKE_CASE` (`MAX_RETRIES`)
- Tipos e interfaces: `PascalCase` (`UserProfileProps`)
- Contextos: `PascalCase` com sufixo `Context` (`AuthContext/`)
- Providers: `PascalCase` com sufixo `Provider` (`AuthProvider/`)

### Estrutura de Módulos Internos

O padrão de pasta se aplica a **componentes, hooks e utils** — qualquer módulo que cresça o suficiente para justificar separação de responsabilidades.

**Componentes:**
```
Button/
  index.tsx          ← componente principal (ponto de entrada, tipos pequenos e não-compartilhados ficam aqui)
  styles.ts          ← estilização (ou .css/.scss conforme a UI library)
  index.test.tsx     ← testes do componente
  types.ts           ← apenas se interfaces/types forem grandes ou compartilhadas com outros componentes
```

**Hooks e Utils (quando justificam pasta):**
```
useAuth/
  index.ts           ← implementação principal
  index.test.ts      ← testes
  types.ts           ← apenas se necessário
```

**Hooks e Utils simples (arquivo único):**
```
hooks/
  useDebounce.ts     ← hook simples que não precisa de pasta própria
utils/
  formatDate.ts      ← utilitário simples que não precisa de pasta própria
```

> Use pasta quando o módulo tem testes, tipos separados, ou múltiplos arquivos auxiliares.
> Use arquivo único quando a implementação é pequena e autocontida.

**Regras gerais:**
- `index.tsx` / `index.ts` é o único export público — importações externas usam o path da pasta (`import { Button } from '@/components/Button'`)
- `types.ts` só deve existir quando os tipos são volumosos ou reutilizados por outros módulos — para tipos simples, mantenha no próprio `index`
- Proibido importar paths internos de outro módulo (ex: `../Button/styles`, `../useAuth/types`)

### Arquitetura de Pastas

A estrutura base segue o padrão abaixo. Pode variar conforme a stack (Next.js usa `app/` em vez de `pages/`, por exemplo), mas a separação de responsabilidades é a mesma:

```
src/
├── components/     # Componentes verdadeiramente genéricos e reutilizáveis
│   ├── Button/
│   ├── Modal/
│   └── Input/
├── pages/          # Páginas da aplicação
├── routes/         # Rotas da aplicação
├── hooks/          # Hooks verdadeiramente globais
├── utils/          # Funções utilitárias puras
└── types/          # Types/interfaces globais e compartilhadas
```

- `components/` — apenas componentes genéricos reutilizáveis (UI primitivos, layout). Componentes específicos de uma feature vivem na pasta da feature/página.
- `hooks/` — apenas hooks globais. Hooks específicos de um componente vivem na pasta do componente.
- `types/` — apenas types/interfaces compartilhadas entre múltiplos módulos. Types locais ficam no componente.
- Barrel exports (`index.ts`) em cada módulo — proibido importar paths internos de outro módulo

### TypeScript

- `strict: true` habilitado — sem exceções
- Proibido `any` — use `unknown` quando o tipo é realmente indeterminado
- Proibido type assertions (`as`) salvo em integrações com libs sem tipagem
- Interfaces para props, types para unions e utilitários
- Retorno de funções explícito em funções públicas — inferência apenas em funções internas

### Boas Práticas React

- Componentes são arrow functions (`const Component = () => {}`) — proibido class components e function declarations
- Proibido lógica de negócio dentro do JSX — extraia para variáveis ou hooks
- Proibido `useEffect` para derivar estado — use `useMemo` ou compute diretamente
- Proibido `useEffect` para sincronizar com props — remodele com key ou estado elevado
- Callbacks estáveis: envolva com `useCallback` apenas quando passados como prop a componentes memoizados
- Proibido `index` como key em listas dinâmicas (renderização, reordenação)
- Proibido manipulação direta do DOM — use refs quando necessário

### Imports

- Ordenação automática via ESLint ou Prettier (respeitar config do projeto)
- Imports absolutos preferidos sobre relativos profundos (`@/components/...`)
- Proibido importar side-effects desnecessários (`import 'lib/styles'` sem uso)

---

## Guias Complementares

Para detalhes, exemplos e anti-patterns de cada tema, consulte os arquivos dedicados nesta mesma pasta:

- **Componentes:** [`components.md`](./components.md) — regras, estrutura e exemplos práticos de componentes React
- **Hooks:** [`hooks.md`](./hooks.md) — regras, estrutura e exemplos de hooks customizados
- **Utils:** [`utils.md`](./utils.md) — regras, estrutura e exemplos de funções utilitárias
- **Testes:** [`tests.md`](./tests.md) — framework, abordagem, hierarquia de queries e exemplos por cenário

---

## Acessibilidade [PADRÃO]

- Navegação por teclado funcional em todos os fluxos principais
- Imagens têm `alt` descritivo — `alt=""` apenas para imagens decorativas

---

## Performance [PADRÃO]

- Proibido importar bibliotecas inteiras quando só parte é usada (`import { X } from 'lib'`)
- Lazy loading para rotas e componentes pesados
- Imagens com dimensões explícitas para evitar layout shift
