# Fluxos de Perguntas

> Arquivo de referência compartilhado entre `sdd.setup`, `sdd.evolve` e `sdd.adopt`.
> Contém as opções canônicas de `AskUserQuestion` para cada campo de stack.
> (O `sdd.adopt` usa estes blocos apenas no preenchimento de lacunas — campos não detectados —
> e na confirmação do modo de arquitetura; valores fora do catálogo são texto livre.)
>
> **Convenção:** Seções sob blocos marcados com `[PROJETO]` aceitam modo "Manter:"
> quando usadas pelo `sdd.evolve` em modo de modificação. Nesse modo, a opção que
> corresponde ao valor atual é substituída por `Manter: <valor atual>` com description
> "Não alterar".
>
> **Campos fundamentais (imutáveis no `sdd.evolve`):** `### Framework` (Frontend);
> `### Linguagem`, `### Runtime — *`, `### Framework — *` (Backend); `### Banco principal`
> (Database). Esses blocos são usados pelo `sdd.setup` e pelo fluxo de **adição** do
> `sdd.evolve`, mas **nunca** entram no modo "Manter" da modificação — ver "Campos
> Fundamentais" em `modification-pattern.md`.

---

## Seleção de Camadas

> Bloco canônico de seleção de camadas de arquitetura. Usado por `sdd.setup` (Fase 2) e
> `sdd.evolve` (Fases 2A/2B). No `sdd.evolve`, filtre dinamicamente (só camadas não ativas
> em 2A; só ativas em 2B) e, em 2B, acrescente o resumo da stack atual na `description`.

```
questions: [
  {
    question: "Quais camadas se aplicam a este projeto?",
    header: "Arquitetura",
    multiSelect: true,
    options: [
      { label: "Frontend",  description: "Interface web: SPA ou SSR" },
      { label: "Backend",   description: "Servidor, API REST/GraphQL ou serviços" },
      { label: "Database",  description: "Banco de dados (relacional ou NoSQL)" },
      { label: "DevOps",    description: "CI/CD, containers, deploy e infraestrutura" }
    ]
  }
]
```

---

## Modo de Arquitetura

> Bloco canônico do modo de arquitetura. Usado por `sdd.setup` (Fase 3) e `sdd.evolve`
> (Fase 3). Aplica-se apenas quando Frontend **e** Backend coexistem.

```
questions: [
  {
    question: "Como o frontend e o backend se relacionam neste projeto?",
    header: "Arquitetura",
    options: [
      {
        label: "Separados",
        description: "Stacks independentes — cada camada com seu próprio repositório, deploy ou processo de build"
      },
      {
        label: "Integrado",
        description: "Frontend servido pelo backend — mesma base de código, sem API separada obrigatória (ex: Laravel + Inertia)"
      }
    ]
  }
]
```

- **Separados:** Frontend e Backend são tratados de forma independente (seção Frontend padrão).
- **Integrado:** execute a seção **Backend** primeiro e, ao terminar, use a seção
  **Frontend Integrado**. Pule a seção Frontend padrão.

---

## Frontend [PROJETO]

### Framework

```
questions: [
  {
    question: "Qual o framework principal?",
    header: "Framework",
    options: [
      { label: "React 18 + TypeScript",  description: "SPA com tipagem estrita" },
      { label: "Next.js 14",             description: "Full-stack com SSR/SSG e App Router" }
    ]
  }
]
```

> Se o usuário escolher "Outra", pergunte estilização, estado global, estado de servidor,
> formulários e roteamento em texto livre antes de continuar.

### Estado global

```
questions: [
  {
    question: "Qual a solução de estado global?",
    header: "Estado global",
    options: [
      { label: "Zustand",       description: "Leve e sem boilerplate" },
      { label: "Redux Toolkit", description: "Robusto com DevTools e middleware" },
      { label: "Jotai",         description: "Atômico, minimal, sem contexto global" },
      { label: "Não se aplica", description: "Context API ou sem estado global centralizado" }
    ]
  }
]
```

### Estado de servidor

```
questions: [
  {
    question: "Como será gerenciado o estado de servidor?",
    header: "Estado servidor",
    options: [
      { label: "TanStack Query",  description: "Cache, sync e background refetch" },
      { label: "SWR",             description: "Simples, focado em fetch e cache" },
      { label: "Apollo Client",   description: "Para GraphQL com cache normalizado" },
      { label: "Não se aplica",   description: "Fetch manual ou sem estado remoto" }
    ]
  }
]
```

### Estilização

```
questions: [
  {
    question: "Como será feita a estilização?",
    header: "Estilização",
    options: [
      { label: "Tailwind CSS",      description: "Utility-first, sem CSS customizado" },
      { label: "CSS Modules",       description: "Escopo local por componente" },
      { label: "Styled Components", description: "CSS-in-JS com props dinâmicas" },
      { label: "Sass / SCSS",       description: "CSS com variáveis e mixins" }
    ]
  }
]
```

### Formulários

```
questions: [
  {
    question: "Qual a biblioteca de formulários?",
    header: "Formulários",
    options: [
      { label: "React Hook Form",  description: "Leve, baseado em refs, sem re-renders" },
      { label: "Formik",           description: "Com validação integrada e abstração" },
      { label: "TanStack Form",    description: "Type-safe, headless, zero deps" },
      { label: "Não se aplica",    description: "Formulários simples sem biblioteca" }
    ]
  }
]
```

### Roteamento — React 18

```
questions: [
  {
    question: "Qual a solução de roteamento?",
    header: "Roteamento",
    options: [
      { label: "React Router v6",  description: "Roteamento declarativo e componível" },
      { label: "TanStack Router",  description: "Type-safe, file-based routing" },
      { label: "Wouter",           description: "Leve, API similar ao React Router" },
      { label: "Não se aplica",    description: "Single page sem roteamento dedicado" }
    ]
  }
]
```

### Roteamento — Next.js 14

```
questions: [
  {
    question: "Qual a solução de roteamento?",
    header: "Roteamento",
    options: [
      { label: "App Router (nativo)",  description: "File-based routing integrado ao Next.js 14" },
      { label: "Pages Router",         description: "Router legado do Next.js, ainda suportado" },
      { label: "Não se aplica",        description: "Single page sem roteamento dedicado" }
    ]
  }
]
```

### UI Library

> Apenas quando framework é React 18 ou Next.js 14 **e** estilização é Tailwind CSS.

```
questions: [
  {
    question: "Qual biblioteca de componentes UI?",
    header: "UI Library",
    options: [
      { label: "shadcn/ui",
        description: "Componentes copy-paste sobre Radix UI + Tailwind — mais popular atualmente" },
      { label: "Headless UI",
        description: "Primitivos acessíveis sem estilização, pela equipe do Tailwind" },
      { label: "DaisyUI",
        description: "Componentes prontos como classes Tailwind — sem JavaScript adicional" },
      { label: "Não se aplica",
        description: "Componentes 100% customizados — sem biblioteca de UI" }
    ]
  }
]
```

> Se o usuário escolher "Outra", aceitar texto livre. Como o passo é condicionado a Tailwind CSS,
> prefira sugerir libs do ecossistema Tailwind (ex: Flowbite, Preline, Tremor).

---

## Backend [PROJETO]

### Linguagem

```
questions: [
  {
    question: "Qual a linguagem principal do backend?",
    header: "Linguagem",
    options: [
      { label: "TypeScript / JavaScript",  description: "Ecossistema Node.js ou Deno" },
      { label: "PHP",                      description: "Laravel, Symfony, etc." }
    ]
  }
]
```

> Se o usuário escolher "Outra", pergunte runtime, framework, ORM, autenticação e validação em texto livre.

### Runtime — TS/JS

```
questions: [
  {
    question: "Qual o runtime?",
    header: "Runtime",
    options: [
      { label: "Node.js 20",  description: "LTS atual, ecossistema mais amplo" },
      { label: "Node.js 22",  description: "LTS mais recente" },
      { label: "Deno 2",      description: "Seguro por padrão, suporte nativo a TS" }
    ]
  }
]
```

### Runtime — PHP

```
questions: [
  {
    question: "Qual o runtime?",
    header: "Runtime",
    options: [
      { label: "PHP 8.3",     description: "Versão estável atual" },
      { label: "PHP 8.2",     description: "Versão LTS anterior" },
      { label: "PHP 8.4",     description: "Versão mais recente" },
      { label: "FrankenPHP",  description: "Runtime moderno embutido no Caddy" }
    ]
  }
]
```

### Framework — TS/JS

```
questions: [
  {
    question: "Qual o framework de servidor?",
    header: "Framework",
    options: [
      { label: "Fastify",   description: "Alto desempenho, schema-first" },
      { label: "NestJS",    description: "Estruturado, decorators, DI nativo" },
      { label: "Express",   description: "Minimalista, ecossistema enorme" },
      { label: "Hono",      description: "Ultra leve, edge-ready" }
    ]
  }
]
```

### Framework — PHP

```
questions: [
  {
    question: "Qual o framework de servidor?",
    header: "Framework",
    options: [
      { label: "Laravel",   description: "Full-stack, DX elevada, Eloquent + Blade" },
      { label: "Symfony",   description: "Robusto, componentizado, enterprise" },
      { label: "Slim",      description: "Micro-framework, leve e flexível" }
    ]
  }
]
```

### ORM — TS/JS

```
questions: [
  {
    question: "Qual o ORM ou query builder?",
    header: "ORM",
    options: [
      { label: "Drizzle ORM",   description: "Type-safe, próximo ao SQL" },
      { label: "Prisma",        description: "DX elevada, schema declarativo" },
      { label: "Knex",          description: "Query builder leve e flexível" },
      { label: "Não se aplica", description: "Queries manuais ou sem ORM" }
    ]
  }
]
```

### ORM — PHP

```
questions: [
  {
    question: "Qual o ORM ou query builder?",
    header: "ORM",
    options: [
      { label: "Eloquent",      description: "Nativo do Laravel, ActiveRecord" },
      { label: "Doctrine",      description: "DataMapper, padrão do Symfony" },
      { label: "Cycle ORM",     description: "Moderno, suporte a relacionamentos complexos" },
      { label: "Não se aplica", description: "Queries manuais ou sem ORM" }
    ]
  }
]
```

### Autenticação — TS/JS

```
questions: [
  {
    question: "Como será feita a autenticação?",
    header: "Autenticação",
    options: [
      { label: "JWT + refresh token",  description: "Stateless, tokens rotativos" },
      { label: "OAuth2",               description: "Delegação de acesso com providers" },
      { label: "Lucia Auth",           description: "Biblioteca moderna de sessão + token" },
      { label: "Não se aplica",        description: "Sem autenticação ou a definir" }
    ]
  }
]
```

### Autenticação — PHP

```
questions: [
  {
    question: "Como será feita a autenticação?",
    header: "Autenticação",
    options: [
      { label: "Laravel Sanctum",   description: "Tokens de API e sessão para SPAs" },
      { label: "Laravel Passport",  description: "OAuth2 completo para Laravel" },
      { label: "JWT",               description: "Stateless com tymon/jwt-auth" },
      { label: "Não se aplica",     description: "Sem autenticação ou a definir" }
    ]
  }
]
```

### Validação — TS/JS

```
questions: [
  {
    question: "Qual a biblioteca de validação?",
    header: "Validação",
    options: [
      { label: "Zod",              description: "Type-safe, inferência de tipos automática" },
      { label: "Joi",              description: "Madura, amplamente adotada" },
      { label: "class-validator",  description: "Decorators, boa integração com NestJS" },
      { label: "Não se aplica",    description: "Validação manual ou a definir" }
    ]
  }
]
```

### Validação — PHP

```
questions: [
  {
    question: "Qual a biblioteca de validação?",
    header: "Validação",
    options: [
      { label: "Laravel Validator",   description: "Nativo do Laravel, fluente" },
      { label: "Symfony Validator",   description: "Baseado em constraints, annotations" },
      { label: "Respect/Validation",  description: "Standalone, fluent interface" },
      { label: "Não se aplica",       description: "Validação manual ou a definir" }
    ]
  }
]
```

---

## Frontend Integrado [PROJETO]

> Esta seção se aplica ao modo **Integrado** (Frontend + Backend no mesmo repositório).
> As opções dependem do framework Backend escolhido.

### Laravel

```
questions: [
  {
    question: "Como o frontend será integrado ao Laravel?",
    header: "Frontend",
    options: [
      { label: "Blade + Alpine.js",    description: "Templates server-side com interatividade leve, sem SPA" },
      { label: "Livewire",             description: "Componentes reativos server-side, sem JavaScript pesado" },
      { label: "Inertia.js + React",   description: "SPA com routing e auth do Laravel, sem API REST separada" }
    ]
  }
]
```

### Symfony

```
questions: [
  {
    question: "Como o frontend será integrado ao Symfony?",
    header: "Frontend",
    options: [
      { label: "Twig + Stimulus",      description: "Templates nativos com Symfony UX" },
      { label: "Symfony UX + Turbo",   description: "Navegação rápida com Turbo e Stimulus" },
      { label: "Inertia.js + React",   description: "SPA com routing do Symfony, sem API REST separada" }
    ]
  }
]
```

### Slim

```
questions: [
  {
    question: "Como o frontend será integrado ao Slim?",
    header: "Frontend",
    options: [
      { label: "Twig",                 description: "Templates server-side via slim/twig-view" },
      { label: "PHP templates",        description: "Views PHP simples, sem engine de template" },
      { label: "HTMX",                 description: "Interatividade hypermedia sem SPA" }
    ]
  }
]
```

> Slim é um micro-framework sem stack SPA integrada — todas as opções são server-side,
> sem Passo 2 de stack complementar.

### Node.js (Fastify / Express / NestJS / Hono)

```
questions: [
  {
    question: "Como o frontend será integrado ao servidor Node.js?",
    header: "Frontend",
    options: [
      { label: "EJS / Handlebars",     description: "Templates server-side clássicos" },
      { label: "HTMX",                 description: "Interatividade hypermedia sem SPA" },
      { label: "Inertia.js + React",   description: "SPA sem API separada" }
    ]
  }
]
```

### Stack complementar (Inertia.js + React)

> Se **Inertia.js + React**: execute os passos de Estado global, Estado de servidor,
> Estilização e Formulários da seção Frontend. Para roteamento, use "Não se aplica"
> — o routing é gerenciado pelo backend. Em seguida, aplique a etapa **UI Library**
> (condicional): se a estilização escolhida for Tailwind CSS, pergunte `### UI Library`;
> senão, pule.
>
> Para todas as demais abordagens (Blade, Livewire, Twig, HTMX, Templates, etc.),
> não há stack de SPA a configurar.

---

## Database [PROJETO]

### Banco principal

```
questions: [
  {
    question: "Qual o banco de dados principal?",
    header: "Banco principal",
    options: [
      { label: "PostgreSQL 16",  description: "Relacional, robusto e extensível" },
      { label: "MySQL 8",        description: "Relacional, amplamente adotado" },
      { label: "MongoDB",        description: "NoSQL orientado a documentos" },
      { label: "SQLite",         description: "Arquivo local, ideal para dev/testes" }
    ]
  }
]
```

### Banco de desenvolvimento

```
questions: [
  {
    question: "Como será o banco em desenvolvimento?",
    header: "Dev DB",
    options: [
      { label: "Docker local",               description: "Instância isolada via container" },
      { label: "SQLite local",               description: "Arquivo simples, sem instalação" },
      { label: "Banco remoto compartilhado", description: "Instância de dev compartilhada" },
      { label: "Não se aplica",              description: "Banco embutido ou não gerenciado" }
    ]
  }
]
```

### Migrations — TS/JS

> Use quando o Backend usa TypeScript / JavaScript.

```
questions: [
  {
    question: "Como serão gerenciadas as migrations?",
    header: "Migrations",
    options: [
      { label: "Drizzle Kit",     description: "Integrado ao Drizzle ORM" },
      { label: "Prisma Migrate",  description: "Integrado ao Prisma" },
      { label: "Knex migrations", description: "Integrado ao Knex" },
      { label: "Não se aplica",   description: "Sem migrations automatizadas" }
    ]
  }
]
```

### Migrations — PHP

> Use quando o Backend usa PHP.

```
questions: [
  {
    question: "Como serão gerenciadas as migrations?",
    header: "Migrations",
    options: [
      { label: "Laravel Migrations",  description: "Nativo do Laravel" },
      { label: "Doctrine Migrations", description: "Padrão do Symfony" },
      { label: "Phinx",               description: "Agnóstico de framework" },
      { label: "Não se aplica",       description: "Sem migrations automatizadas" }
    ]
  }
]
```

### ORM + Migrations — Sem Backend ativo

> Use este bloco quando Database está ativo mas Backend **não**.
> Pergunta ORM + Migrations juntos.

```
questions: [
  {
    question: "Qual o ORM ou query builder?",
    header: "ORM",
    options: [
      { label: "Drizzle ORM / Prisma",  description: "TypeScript — type-safe" },
      { label: "Eloquent / Doctrine",   description: "PHP — Laravel ou Symfony" },
      { label: "Não se aplica",         description: "Queries manuais ou sem ORM" }
    ]
  },
  {
    question: "Como serão gerenciadas as migrations?",
    header: "Migrations",
    options: [
      { label: "Drizzle Kit / Prisma Migrate", description: "TypeScript" },
      { label: "Laravel / Doctrine Migrations", description: "PHP" },
      { label: "Flyway / Liquibase",           description: "Agnóstico de linguagem" },
      { label: "Não se aplica",                description: "Sem migrations automatizadas" }
    ]
  }
]
```

### Banco de teste

```
questions: [
  {
    question: "Como será o banco nos testes?",
    header: "Test DB",
    options: [
      { label: "Docker Compose isolado",     description: "Container dedicado por pipeline" },
      { label: "SQLite em memória",          description: "Rápido, sem persistência" },
      { label: "Banco real em amb. isolado", description: "Instância separada de staging" },
      { label: "Não se aplica",              description: "Sem testes de integração com banco" }
    ]
  }
]
```

---

## DevOps [PROJETO]

### CI/CD, Containers, Hospedagem, Monitoramento

```
questions: [
  {
    question: "Qual a plataforma de CI/CD?",
    header: "CI/CD",
    options: [
      { label: "GitHub Actions",  description: "Nativo ao GitHub, configurado em YAML" },
      { label: "GitLab CI",       description: "Nativo ao GitLab" },
      { label: "CircleCI",        description: "Plataforma dedicada de CI" },
      { label: "Não se aplica",   description: "Sem pipeline automatizado" }
    ]
  },
  {
    question: "Haverá containerização?",
    header: "Containers",
    options: [
      { label: "Docker + Docker Compose",  description: "Padrão de mercado" },
      { label: "Podman",                   description: "Compatível com Docker, rootless" },
      { label: "Kubernetes",               description: "Orquestração em produção" },
      { label: "Não se aplica",            description: "Sem containerização" }
    ]
  },
  {
    question: "Onde será hospedada a aplicação?",
    header: "Hospedagem",
    options: [
      { label: "Railway / Fly.io",    description: "PaaS moderno, baixo overhead" },
      { label: "Vercel / Netlify",    description: "Foco em frontend e edge functions" },
      { label: "AWS / GCP / Azure",   description: "Cloud provider completo" },
      { label: "Não se aplica",       description: "On-premise ou a definir" }
    ]
  },
  {
    question: "Qual a solução de monitoramento?",
    header: "Monitoramento",
    options: [
      { label: "Sentry",                 description: "Rastreamento de erros e performance" },
      { label: "Datadog",                description: "Observabilidade completa" },
      { label: "Grafana + Prometheus",   description: "Open-source, self-hosted" },
      { label: "Não se aplica",          description: "Sem monitoramento configurado" }
    ]
  }
]
```

### Registry de imagens

```
questions: [
  {
    question: "Onde serão armazenadas as imagens Docker?",
    header: "Image registry",
    options: [
      { label: "GitHub Container Registry",  description: "Integrado ao GitHub" },
      { label: "Docker Hub",                 description: "Registry público padrão" },
      { label: "AWS ECR",                    description: "Integrado ao ecossistema AWS" },
      { label: "Não se aplica",              description: "Sem registry de imagens" }
    ]
  }
]
```

---

## Mapeamento Rápido: Campo → Seção

> **Tabela canônica** da relação campo → seção → `header` do `AskUserQuestion`.
> Usada tanto pelo fluxo de adição (`addition-flow.md`) quanto pelo de modificação
> (`modification-pattern.md`) — não duplicar esta tabela em outros arquivos.
> A coluna **Header** é o valor literal do campo `header` ao montar o `AskUserQuestion`.

| Campo | Arquivo | Seção | Header |
|-------|---------|-------|--------|
| Camadas ativas | — | Seleção de Camadas | `Arquitetura` |
| Modo de arquitetura | — | Modo de Arquitetura | `Arquitetura` |
| Framework | Frontend | Framework | `Framework` |
| Estado global | Frontend | Estado global | `Estado global` |
| Estado de servidor | Frontend | Estado de servidor | `Estado servidor` |
| Estilização | Frontend | Estilização | `Estilização` |
| Formulários | Frontend | Formulários | `Formulários` |
| Roteamento (React) | Frontend | Roteamento — React 18 | `Roteamento` |
| Roteamento (Next.js) | Frontend | Roteamento — Next.js 14 | `Roteamento` |
| UI Library | Frontend | UI Library | `UI Library` |
| Frontend Integrado (Laravel) | Frontend Integrado | Laravel | `Frontend` |
| Frontend Integrado (Symfony) | Frontend Integrado | Symfony | `Frontend` |
| Frontend Integrado (Slim) | Frontend Integrado | Slim | `Frontend` |
| Frontend Integrado (Node.js) | Frontend Integrado | Node.js | `Frontend` |
| Linguagem | Backend | Linguagem | `Linguagem` |
| Runtime (JS) | Backend | Runtime — TS/JS | `Runtime` |
| Runtime (PHP) | Backend | Runtime — PHP | `Runtime` |
| Framework (JS) | Backend | Framework — TS/JS | `Framework` |
| Framework (PHP) | Backend | Framework — PHP | `Framework` |
| ORM (JS) | Backend | ORM — TS/JS | `ORM` |
| ORM (PHP) | Backend | ORM — PHP | `ORM` |
| Autenticação (JS) | Backend | Autenticação — TS/JS | `Autenticação` |
| Autenticação (PHP) | Backend | Autenticação — PHP | `Autenticação` |
| Validação (JS) | Backend | Validação — TS/JS | `Validação` |
| Validação (PHP) | Backend | Validação — PHP | `Validação` |
| Banco principal | Database | Banco principal | `Banco principal` |
| Dev DB | Database | Banco de desenvolvimento | `Dev DB` |
| Migrations (JS) | Database | Migrations — TS/JS | `Migrations` |
| Migrations (PHP) | Database | Migrations — PHP | `Migrations` |
| ORM + Migrations (sem Backend) | Database | ORM + Migrations — Sem Backend ativo | `ORM` / `Migrations` |
| Test DB | Database | Banco de teste | `Test DB` |
| CI/CD | DevOps | CI/CD, Containers, Hospedagem, Monitoramento | `CI/CD` |
| Containers | DevOps | CI/CD, Containers, Hospedagem, Monitoramento | `Containers` |
| Hospedagem | DevOps | CI/CD, Containers, Hospedagem, Monitoramento | `Hospedagem` |
| Monitoramento | DevOps | CI/CD, Containers, Hospedagem, Monitoramento | `Monitoramento` |
| Registry | DevOps | Registry de imagens | `Image registry` |
