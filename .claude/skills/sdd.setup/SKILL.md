---
name: sdd.setup
description: Configura um novo projeto preenchendo todos os blocos [PROJETO] no CLAUDE.md e nos guias de arquitetura em .specs/architecture. Use quando o usuário iniciar um novo projeto, copiar os templates ou pedir setup do SDD.
disable-model-invocation: true
model: opus
effort: high
---

## O que esse skill faz

Coleta as informações específicas do projeto através de um questionário guiado e substitui todos os
blocos `[PROJETO]` nos arquivos de template:

- `CLAUDE.md`
- `.specs/architecture/*/spec.md` (apenas os guias marcados como ativos)

Antes de começar, leia os arquivos listados acima para identificar quais blocos `[PROJETO]` ainda
não foram preenchidos. Se o projeto já estiver parcialmente configurado, pule as fases já concluídas.

---

## Fase 1 — Identidade do Projeto

Pergunte em texto (entrada livre, uma pergunta por vez):

1. **Nome do projeto** — nome literal, como o usuário chama o projeto (ex: "Strava Dashboard", "API de Pagamentos", "Portal do Cliente")
2. **Descrição curta** — 1 linha descrevendo o que é o projeto

Após receber o nome literal, gere um slug (lowercase, palavras separadas por hífen, sem caracteres
especiais) e confirme com o usuário antes de prosseguir.
Exemplo: "Strava Dashboard" → `strava-dashboard`.

O slug será usado no campo `Nome` do `CLAUDE.md`.

> A stack principal será derivada automaticamente das respostas da Fase 3.

---

## Fase 2 — Guias de Arquitetura Ativos

Use a ferramenta `AskUserQuestion` com os seguintes parâmetros:

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

## Fase 2.5 — Modo de Arquitetura

> Esta fase só se aplica quando **Frontend** e **Backend** foram ambos selecionados na Fase 2.
> Se apenas um deles foi selecionado, pule diretamente para a Fase 3.

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

- Se **Separados**: prossiga normalmente para a Fase 3 — Frontend e Backend são tratados de forma independente.
- Se **Integrado**: na Fase 3, execute a seção **Backend** primeiro e, ao terminar, use a seção **Frontend Integrado**. Pule a seção Frontend padrão.

---

## Fase 3 — Stack dos Guias Ativos

Para cada guia selecionado na Fase 2, faça chamadas à ferramenta `AskUserQuestion` agrupando os
campos (máximo de 4 perguntas por chamada). Execute os blocos de um guia de cada vez.

O usuário pode selecionar uma das opções apresentadas ou escolher "Outra" para informar um valor
personalizado.

### Frontend

> Esta seção se aplica apenas ao modo **Separados** (Fase 2.5).
> Se o modo for **Integrado**, pule para a seção **Frontend Integrado** após concluir o Backend.

O Frontend usa um fluxo condicional: primeiro pergunta o framework e, com base na resposta,
apresenta opções de estado, estilização, formulários e roteamento filtradas para aquele ecossistema.

**Passo 1 — Framework** (chamada isolada):
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

---

**Passo 2 — Estado global e Estado de servidor** (2 campos)

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
  },
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

---

**Passo 3 — Estilização, Formulários e Roteamento** (3 campos)

Estilização e Formulários (chamada compartilhada):
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
  },
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

Em seguida, para o roteamento (chamada separada, condicional pelo framework):

Se **React 18 + TypeScript**:
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

Se **Next.js 14**:
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

**Passo 4 — UI Library** (chamada separada, 1 campo)

Pergunte apenas se o framework for React 18 + TypeScript ou Next.js 14, e se a
estilização for Tailwind CSS.

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
      { label: "Nenhuma",
        description: "Componentes 100% customizados — sem biblioteca de UI" }
    ]
  }
]
```

Se o usuário escolher "Outra", aceitar texto livre (ex: NextUI, Mantine, Chakra UI).

> Esta pergunta é opcional — se a estilização não for Tailwind CSS, pule.

### Frontend Integrado

> Esta seção se aplica apenas ao modo **Integrado** (Fase 2.5), quando Frontend e Backend foram selecionados.
> Execute-a após concluir toda a seção Backend.

Com base no framework Backend escolhido, apresente as abordagens de frontend compatíveis.

**Passo 1 — Abordagem de integração** (chamada condicional pelo framework Backend):

Se **Laravel**:
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

Se **Symfony**:
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

Se **Fastify**, **Express**, **NestJS** ou **Hono** (Node.js):
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

---

**Passo 2 — Stack complementar** (somente se escolheu **Inertia.js + React**)

Se **Inertia.js + React**: execute os Passos 2 e 3 da seção Frontend padrão (estado global,
estado de servidor, estilização, formulários). Para roteamento, use "Não se aplica" — o routing
é gerenciado pelo backend.

Para todas as demais abordagens (Blade, Livewire, Twig, HTMX, Templates, etc.), pule o Passo 2
— não há stack de SPA a configurar.

### Backend

O Backend usa um fluxo condicional: primeiro pergunta a linguagem e, com base na resposta, apresenta
opções de runtime, framework, ORM e validação filtradas para aquele ecossistema.

**Passo 1 — Linguagem** (chamada isolada):
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

> Se o usuário escolher "Outra", pergunte runtime, framework, ORM e validação em texto livre
> antes de continuar com o Passo 3.

---

**Passo 2 — Runtime, Framework e ORM** (chamada condicional com 3 campos)

Use as opções abaixo de acordo com a linguagem selecionada no Passo 1:

**Se TypeScript / JavaScript:**
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
  },
  {
    question: "Qual o framework de servidor?",
    header: "Framework",
    options: [
      { label: "Fastify",   description: "Alto desempenho, schema-first" },
      { label: "NestJS",    description: "Estruturado, decorators, DI nativo" },
      { label: "Express",   description: "Minimalista, ecossistema enorme" },
      { label: "Hono",      description: "Ultra leve, edge-ready" }
    ]
  },
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

**Se PHP:**
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
  },
  {
    question: "Qual o framework de servidor?",
    header: "Framework",
    options: [
      { label: "Laravel",   description: "Full-stack, DX elevada, Eloquent + Blade" },
      { label: "Symfony",   description: "Robusto, componentizado, enterprise" },
      { label: "Slim",      description: "Micro-framework, leve e flexível" }
    ]
  },
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

---

**Passo 3 — Autenticação e Validação** (chamada condicional com 2 campos)

Use as opções abaixo de acordo com a linguagem selecionada no Passo 1:

**Se TypeScript / JavaScript:**
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
  },
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

**Se PHP:**
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
  },
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

### Database

**Chamada 1** (2 campos — banco e ambiente de desenvolvimento):
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
  },
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

**Chamada 2 — Migrations** (condicional pela linguagem do Backend)

> Se o Backend está ativo, o ORM já foi coletado naquela seção — não pergunte novamente.
> Pergunte apenas as Migrations, usando as opções condicionais abaixo.
> Se o Backend não está ativo, pergunte ORM + Migrations juntos com o bloco de fallback ao final.

Se Backend ativo com **TypeScript / JavaScript**:
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

Se Backend ativo com **PHP**:
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

Se Backend **não selecionado** ou linguagem **Outra** — pergunte ORM + Migrations:
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

**Chamada 3** (1 campo):
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

### DevOps

**Chamada 1** (4 campos):
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

**Chamada 2** (1 campo):
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

## Fase 4 — Comandos do Projeto

Com base nas camadas ativas e escolhas da Fase 3, derive os comandos por camada usando as tabelas
abaixo. Apresente as sugestões ao usuário organizadas por camada e peça confirmação ou correção
em texto livre.

Após confirmar, componha a lista final para o `CLAUDE.md`. Se uma mesma ação tiver comandos em
camadas diferentes (ex: `composer install` + `npm install`), registre-os em linhas separadas com
comentários indicando a camada. Omita completamente as ações que não se aplicarem.

---

### Frontend (apenas se ativo)

| Ação | Derivação |
|------|-----------|
| Instalar dependências | `npm install` |
| Servidor de dev | Next.js → `npm run dev` · React → `npm run dev` |
| Rodar testes | `npm test` |
| Verificar lint | `npm run lint` |
| Verificar tipos | `npm run typecheck` |
| Build de produção | `npm run build` |

---

### Backend (apenas se ativo)

**Instalar dependências por runtime:**

| Runtime | Comando |
|---------|---------|
| Node.js / Deno | `npm install` |
| PHP | `composer install` |

**Servidor de desenvolvimento por framework:**

| Framework | Comando |
|-----------|---------|
| Laravel | `php artisan serve` |
| Symfony | `symfony server:start` |
| Slim | `php -S localhost:8000 -t public` |
| Node.js (Fastify / Express / NestJS / Hono) | `npm run dev` |

**Demais ações:**

| Ação | Derivação |
|------|-----------|
| Rodar testes | Node.js → `npm test` · Laravel → `php artisan test` · Symfony → `php bin/phpunit` |
| Verificar lint | Node.js → `npm run lint` · Laravel → `./vendor/bin/pint` · Symfony → `./vendor/bin/phpcs` |
| Verificar tipos | TypeScript → `npm run typecheck` · PHP → `./vendor/bin/phpstan analyse` |
| Build de produção | Node.js → `npm run build` · PHP sem assets → _(omita)_ |

> **Nota (modo Integrado):** Se o backend PHP usa frontend JS/TS (Inertia + React, Livewire com Alpine, etc.),
> adicione também os comandos de teste e lint do Frontend: `npm test` e `npm run lint`.

---

### DevOps / Docker (apenas se containerização ativa)

Se Docker/Docker Compose foi selecionado e **há ao menos uma outra camada ativa com servidor de dev**
(Frontend ou Backend), existe um conflito de servidor de desenvolvimento a resolver.

Nesse caso, use `AskUserQuestion` com opções montadas a partir dos comandos já derivados para as
camadas ativas. Substitua os placeholders pelos comandos reais antes de chamar a ferramenta:

```
questions: [
  {
    question: "Docker está ativo junto com <liste as camadas ativas, ex: 'Frontend e Backend'>. Como prefere subir o servidor de desenvolvimento?",
    header: "Servidor de dev",
    options: [
      {
        label: "docker compose up",
        description: "Sobe o ambiente completo via container — todas as camadas em um comando"
      },
      {
        label: "<cmd-backend> + <cmd-frontend>",  ← substitua pelos comandos reais derivados acima
        description: "Cada camada sobe separadamente — mais granular durante desenvolvimento"
      },
      {
        label: "Registrar ambos",
        description: "Documentar as duas opções no CLAUDE.md com comentários explicando cada uma"
      }
    ]
  }
]
```

**Regras para montar as opções:**
- A label da opção B deve conter os comandos reais, ex: `php artisan serve + npm run dev`
- Se apenas Backend está ativo (sem Frontend), omita o comando de frontend da label e da description
- Se apenas Frontend está ativo (sem Backend), omita o comando de backend
- Se Docker está ativo mas nenhuma outra camada tem servidor de dev, use `docker compose up` diretamente, sem perguntar
- Se o usuário escolher "Registrar ambos", registre no `CLAUDE.md` com comentários:
  ```bash
  docker compose up          # ambiente completo (container)
  # ou separadamente:
  <cmd-backend>              # backend
  <cmd-frontend>             # frontend
  ```

---

### Exemplo de composição final (PHP + React + Docker)

```bash
composer install  # dependências PHP
npm install       # dependências frontend
docker compose up # servidor de desenvolvimento
php artisan test  # testes
./vendor/bin/pint # lint PHP
npm run lint      # lint frontend
./vendor/bin/phpstan analyse  # typecheck PHP
npm run typecheck # typecheck frontend
npm run build     # build de produção (assets)
```

---

## Fase 5 — Referências Rápidas

Pergunte em texto (entrada livre): há links adicionais para as Referências Rápidas?
Exemplos: documentação externa, URL da API, design system, Figma, board do projeto.

Não use `AskUserQuestion` nesta fase — aceite qualquer formato que o usuário queira informar.
Se não houver links, pule esta fase.

---

## Confirmação das Respostas

Antes de aplicar qualquer mudança, exiba um resumo de tudo que foi coletado nas fases anteriores.
Use o formato abaixo — omita seções que não foram preenchidas (guias inativos, fases puladas).

```
## Resumo do projeto

**Nome:** <slug>
**Descrição:** <descrição>
**Guias ativos:** <lista dos guias selecionados>
**Modo de arquitetura:** <Separados | Integrado — omitir se apenas Frontend ou apenas Backend ativo>

### Frontend
- Framework: <valor>
- Estilização: <valor>
- Estado global: <valor>
- Estado de servidor: <valor>
- Formulários: <valor>
- Roteamento: <valor>
- UI Library: <valor>

### Backend
- Linguagem: <valor>
- Runtime: <valor>
- Framework: <valor>
- ORM: <valor>
- Autenticação: <valor>
- Validação: <valor>

### Database
- Banco principal: <valor>
- ORM / Migrations: <valor>
- Banco de desenvolvimento: <valor>
- Banco de teste: <valor>

### DevOps
- CI/CD: <valor>
- Containerização: <valor>
- Hospedagem: <valor>
- Monitoramento: <valor>
- Registry de imagens: <valor>

### Comandos
<lista de comandos derivados, um por linha>

### Referências
<links informados, ou "Nenhuma" se não houver>
```

Em seguida, use `AskUserQuestion`:

```
questions: [
  {
    question: "As informações acima estão corretas?",
    header: "Confirmação",
    options: [
      { label: "Confirmar e aplicar", description: "Prosseguir com a configuração dos arquivos" },
      { label: "Reiniciar",           description: "Descartar as respostas e refazer o questionário desde o início" }
    ]
  }
]
```

Se o usuário escolher **Reiniciar**, volte ao início da Fase 1 e refaça todo o questionário.
Se confirmar, prossiga para a seção **Aplicação das Mudanças**.

---

## Aplicação das Mudanças

Após coletar todas as respostas, edite os arquivos um de cada vez:

### CLAUDE.md

1. Remova o bloco de instruções no topo (as linhas com `>` antes do primeiro `---`, incluindo a linha em branco seguinte).
2. Preencha `Identidade do Projeto`:
   - `Nome` → slug confirmado na Fase 1
   - `Descrição curta` → resposta da Fase 1
   - `Stack principal` → resumo derivado das respostas da Fase 3 (ex: "React 18 + TypeScript + Tailwind + shadcn/ui + Node.js + PostgreSQL"). Se a Fase 3 não foi executada, pergunte ao usuário.
3. Na seção `Guias de Arquitetura Ativos`, marque `[x]` nos guias selecionados e deixe `[ ]` nos demais.
4. Na seção `Comandos do Projeto`, substitua a linha placeholder da tabela pelos comandos reais derivados na Fase 4. Use o formato:
   ```
   | Ação | Comando | Camada |
   |------|---------|--------|
   | Instalar dependências | `composer install` | backend |
   | Instalar dependências | `npm install` | frontend |
   | Servidor de dev | `php artisan serve` | backend |
   | ...etc |
   ```
   Cada comando em sua própria linha. A coluna "Camada" identifica a qual camada o comando pertence (frontend, backend, devops, etc.). Omita ações que não se aplicam.
5. Na seção `Referências Rápidas`, adicione os links da Fase 5. Se não houver links, remova apenas o placeholder.

### Guias ativos (.specs/architecture/*/spec.md)

Para cada guia marcado como ativo na Fase 2:

1. Remova o bloco de instruções no topo (as linhas com `>` antes do primeiro `---`).
2. Preencha a seção `Stack [PROJETO]` com as respostas da Fase 3. O título deve ficar `## Stack` (sem `[PROJETO]`).
3. Remova as linhas de campos que o usuário indicou como "não se aplica".

### Arquivos complementares do Backend (.specs/architecture/backend/)

Os arquivos `tests.md`, `services.md` e `api.md` contêm exemplos nas duas linguagens (Node.js/TypeScript e PHP).
Após identificar a linguagem escolhida na Fase 3, **remova os exemplos e referências da linguagem não selecionada**:

**Se a linguagem é TypeScript / JavaScript:**
- Em `tests.md`: remova as seções "Exemplo: Teste unitário de service (PHP/Laravel)" e "Exemplo: Teste de integração de rota (PHP/Laravel)"
- Em `services.md`: remova as seções "Exemplo: Service (PHP/Laravel)" e "Exemplo: Erros tipados (PHP/Laravel)"
- Em `api.md`: remova as seções "Exemplo: Controller (PHP/Laravel)", "Exemplo: Validação (PHP/Laravel — FormRequest)" e "Exemplo: Error handler global (PHP/Laravel)"
- Em `spec.md`: na tabela de Nomenclatura, remova a coluna PHP. Na seção "PHP (quando aplicável)", remova-a inteiramente.

**Se a linguagem é PHP:**
- Em `tests.md`: remova as seções "Exemplo: Teste unitário de service (Node.js/TypeScript)", "Exemplo: Teste de integração de rota (Node.js/TypeScript)" e "Exemplo: Factory para testes"
- Em `services.md`: remova as seções "Exemplo: Service (Node.js/TypeScript)", "Exemplo: Erros tipados (Node.js/TypeScript)" e "Exemplo: Repository (Node.js/TypeScript)"
- Em `api.md`: remova as seções "Exemplo: Controller (Node.js/TypeScript — Fastify)", "Exemplo: Validação (Node.js/TypeScript — Zod)", "Exemplo: Middleware de autenticação (Node.js/TypeScript)" e "Exemplo: Error handler global (Node.js/TypeScript)"
- Em `spec.md`: na tabela de Nomenclatura, remova a coluna "Node.js / TypeScript". Na seção "TypeScript (quando aplicável)", remova-a inteiramente.

**Em ambos os casos:**
- Nos anti-patterns, mantenha apenas os exemplos na linguagem selecionada. Se um anti-pattern tem exemplos nas duas linguagens, remova o da linguagem não selecionada.
- Atualize as tabelas comparativas em `spec.md` (seção Testes) para manter apenas a linha da linguagem selecionada.

### Guias inativos

Não modifique guias que não foram selecionados como ativos na Fase 2.

---

## Confirmação Final

Após aplicar todas as mudanças, exiba um resumo compacto:

- Arquivos modificados
- Blocos `[PROJETO]` preenchidos
- Se ficou algum bloco `[PROJETO]` sem preencher (e o motivo)
