# Fluxo de Adição por Camada

> Arquivo de referência compartilhado entre `sdd.setup` e `sdd.evolve`.
> Fonte canônica da **sequência de adição** de uma camada do zero — quais blocos de
> `question-flows.md` chamar, em que ordem, e quando combinar campos numa só chamada.
> (Paralelo ao `modification-pattern.md`, que é dono do fluxo de **modificação**.)
>
> **Uso no `sdd.setup`:** configurar todas as camadas selecionadas (Fase 4).
> **Uso no `sdd.evolve`:** configurar apenas as camadas **novas** (Fase 4A), aplicando por
> cima os ajustes cross-layer descritos na própria skill.

---

## Ordem de execução

Quando há múltiplas camadas novas, execute nesta ordem para resolver dependências
(ORM e linguagem) antes de quem as consome:

```
Backend → Database → Frontend → DevOps
```

> No modo **Integrado** (Frontend + Backend no mesmo repositório), o Frontend é configurado
> pela seção **Frontend Integrado** em vez da seção Frontend padrão — mantendo a ordem
> `Backend → Database → Frontend Integrado → DevOps` (o Database continua entre Backend e Frontend).

---

## Como ler os ponteiros

Cada passo aponta para uma seção de `question-flows.md` pelo seu **heading real**. Quando um
passo lista **mais de uma seção**, monte **uma única chamada `AskUserQuestion`** com um item
por campo no array `questions`, usando as opções de cada seção citada. Os `header` de cada
campo estão na tabela "Mapeamento Rápido" de `question-flows.md`.

---

## Frontend (modo Separados)

1. **Framework** (chamada isolada): `question-flows.md` > `### Framework`.
   - Se o usuário escolher "Outra", pergunte estilização, estado global, estado de servidor,
     formulários e roteamento em texto livre e pule os passos seguintes.
2. **Estado global + Estado de servidor** (uma chamada combinando os dois campos):
   `question-flows.md` > `### Estado global` e `### Estado de servidor`.
3. **Estilização + Formulários** (uma chamada combinando os dois campos):
   `question-flows.md` > `### Estilização` e `### Formulários`.
4. **Roteamento** (chamada isolada, condicional pelo framework):
   - React 18 → `question-flows.md` > `### Roteamento — React 18`
   - Next.js 14 → `question-flows.md` > `### Roteamento — Next.js 14`
5. **UI Library** (chamada isolada, condicional): apenas se framework for React 18 ou Next.js 14
   **E** estilização for Tailwind CSS → `question-flows.md` > `### UI Library`. Senão, pule.

## Frontend Integrado (modo Integrado)

Execute **após** o Backend e o Database (quando ambos ativos), mantendo a ordem
`Backend → Database → Frontend Integrado → DevOps`. Com base no framework Backend, apresente as
abordagens compatíveis (chamada condicional):

- Laravel → `question-flows.md` > Frontend Integrado > `### Laravel`
- Symfony → `question-flows.md` > Frontend Integrado > `### Symfony`
- Slim → `question-flows.md` > Frontend Integrado > `### Slim`
- Node.js (Fastify / Express / NestJS / Hono) → `question-flows.md` > Frontend Integrado > `### Node.js (Fastify / Express / NestJS / Hono)`

Em seguida aplique a regra da seção **"Stack complementar (Inertia.js + React)"** de
`question-flows.md`: se a abordagem for **Inertia.js + React**, execute os passos de Estado global,
Estado de servidor, Estilização e Formulários do Frontend padrão, use "Não se aplica" para
roteamento e, por fim, aplique a etapa **UI Library** condicional (só se a estilização for Tailwind
CSS → `### UI Library`; senão, pule); para as demais abordagens (Blade, Livewire, Twig, HTMX,
templates, Slim), não há stack de SPA a configurar.

## Backend

1. **Linguagem** (chamada isolada): `question-flows.md` > `### Linguagem`.
   - Se "Outra", pergunte runtime, framework, ORM, autenticação e validação em texto livre.
2. **Runtime + Framework + ORM** (uma chamada combinando os três campos, condicional pela linguagem):
   - TS/JS → `question-flows.md` > `### Runtime — TS/JS`, `### Framework — TS/JS`, `### ORM — TS/JS`
   - PHP → `question-flows.md` > `### Runtime — PHP`, `### Framework — PHP`, `### ORM — PHP`
3. **Autenticação + Validação** (uma chamada combinando os dois campos, condicional pela linguagem):
   - TS/JS → `question-flows.md` > `### Autenticação — TS/JS`, `### Validação — TS/JS`
   - PHP → `question-flows.md` > `### Autenticação — PHP`, `### Validação — PHP`

## Database

1. **Banco principal + Banco de desenvolvimento** (uma chamada combinando os dois campos):
   `question-flows.md` > `### Banco principal` e `### Banco de desenvolvimento`.
2. **ORM / Migrations** (condicional):
   - Se **Backend ativo com linguagem conhecida** (TS/JS ou PHP): o ORM **já foi coletado** no
     Backend (ou será herdado) — **não pergunte ORM de novo**. Pergunte só Migrations:
     `question-flows.md` > `### Migrations — TS/JS` ou `### Migrations — PHP`, conforme a
     linguagem do Backend.
   - Se **Backend ativo com linguagem "Outra"**: herde o ORM já coletado em texto livre no Backend
     e pergunte as Migrations **também em texto livre** — os blocos `### Migrations — TS/JS` e
     `### Migrations — PHP` são específicos de ecossistema e não se aplicam. **Não re-pergunte ORM.**
   - Se **sem Backend**: pergunte ORM + Migrations juntos com
     `question-flows.md` > `### ORM + Migrations — Sem Backend ativo`.
3. **Banco de teste** (chamada isolada): `question-flows.md` > `### Banco de teste`.

## DevOps

1. **CI/CD + Containers + Hospedagem + Monitoramento** (uma chamada com os quatro campos):
   `question-flows.md` > `### CI/CD, Containers, Hospedagem, Monitoramento`.
2. **Registry de imagens** (chamada isolada): `question-flows.md` > `### Registry de imagens`.
