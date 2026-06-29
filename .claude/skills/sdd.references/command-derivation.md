# Derivação de Comandos

> Arquivo de referência compartilhado entre `sdd.setup`, `sdd.evolve` e `sdd.adopt`.
> Contém as tabelas canônicas de derivação de comandos por camada.
>
> **Uso no `sdd.setup`:** Derivar todos os comandos do zero (Fase 5).
> **Uso no `sdd.evolve`:** Re-derivar incrementalmente — comandos de camadas não
> alteradas são mantidos; camadas novas derivados do zero; camadas modificadas
> re-derivados apenas se a mudança afetar comandos (Fase 6).
> **Uso no `sdd.adopt`:** **Fallback** — só use estas tabelas quando o projeto não
> declarar comandos próprios (ver nota abaixo).

---

## Brownfield (`sdd.adopt`): leia os comandos reais primeiro

Em projetos existentes, **não derive comandos por tabela** se o projeto já os declara. Leia
os comandos reais nas fontes mapeadas em **"Comandos — onde achar os comandos reais"** de
`stack-detection.md` (scripts de `package.json`/`composer.json`, `Makefile`, `Taskfile.yml`,
`justfile`, `pyproject.toml`, etc.) e use-os diretamente. As tabelas deste arquivo só entram
como **fallback** quando não há nenhum comando declarado — e, mesmo assim, valide o resultado
contra o runtime detectado.

---

## Regra Geral

Após derivar os comandos, apresente as sugestões ao usuário organizadas por camada
e peça confirmação ou correção em texto livre.

> O **formato de saída** no `CLAUDE.md` (uma linha por comando, coluna "Camada", omissão de
> ações inaplicáveis) é canônico na seção **"Comandos do Projeto"** de `file-application.md`.
> Este arquivo cuida apenas da **derivação** (qual comando para cada stack).

---

## Linguagem ou framework "Outra"

As tabelas abaixo cobrem as stacks oferecidas no `question-flows.md`. Quando o usuário escolhe
**"Outra"** para o Framework do Frontend ou a Linguagem do Backend, nenhuma tabela se aplica —
**não invente comandos**. Derive perguntando ao usuário em texto livre os comandos reais para:

- Instalar dependências
- Servidor de desenvolvimento
- Rodar testes
- Verificar lint
- Verificar tipos (se aplicável)
- Build de produção

Registre apenas as ações que existem na stack escolhida; omita as que não se aplicam.

---

## Frontend

| Ação | Derivação |
|------|-----------|
| Instalar dependências | `npm install` |
| Servidor de dev | `npm run dev` |
| Rodar testes | `npm test` |
| Verificar lint | `npm run lint` |
| Verificar tipos | `npm run typecheck` |
| Build de produção | `npm run build` |

---

## Backend

### Instalar dependências por runtime

| Runtime | Comando |
|---------|---------|
| Node.js | `npm install` |
| Deno | `deno install` |
| PHP | `composer install` |

> **Runtime Deno:** Deno não usa o ecossistema `npm run`. Substitua os comandos das tabelas
> abaixo pelos equivalentes nativos: dev `deno task dev` · testes `deno test` ·
> lint `deno lint` · tipos `deno check` · build `deno task build`.

### Servidor de desenvolvimento por framework

| Framework | Comando |
|-----------|---------|
| Laravel | `php artisan serve` |
| Symfony | `symfony server:start` |
| Slim | `php -S localhost:8000 -t public` |
| Node.js (Fastify / Express / NestJS / Hono) | `npm run dev` |

### Demais ações

| Ação | Derivação |
|------|-----------|
| Rodar testes | Node.js → `npm test` · Laravel → `php artisan test` · Symfony → `php bin/phpunit` |
| Verificar lint | Node.js → `npm run lint` · Laravel → `./vendor/bin/pint` · Symfony → `./vendor/bin/phpcs` |
| Verificar tipos | TypeScript → `npm run typecheck` · PHP → `./vendor/bin/phpstan analyse` |
| Build de produção | Node.js → `npm run build` · PHP sem assets → _(omita)_ |

> **Nota (modo Integrado):** Se o backend PHP usa frontend JS/TS (Inertia + React,
> Livewire com Alpine, etc.), adicione também os comandos de Frontend que se aplicam à stack:
> `npm test` e `npm run lint` sempre; e, quando há build de assets (ex: Inertia + React),
> também `npm run typecheck` e `npm run build`.

---

## DevOps / Docker

> Aplicável apenas se containerização (Docker/Docker Compose, Podman, Kubernetes)
> foi selecionada no DevOps.

### Conflito de servidor de dev

Se Docker/Docker Compose foi selecionado e **há ao menos uma outra camada ativa com
servidor de dev** (Frontend ou Backend), existe um conflito a resolver.

Use `AskUserQuestion` com opções montadas a partir dos comandos já derivados para as
camadas ativas. Substitua os placeholders pelos comandos reais:

```
questions: [
  {
    question: "Docker está ativo junto com <liste as camadas ativas>. Como prefere subir o servidor de desenvolvimento?",
    header: "Servidor de dev",
    options: [
      {
        label: "docker compose up",
        description: "Sobe o ambiente completo via container — todas as camadas em um comando"
      },
      {
        label: "<cmd-backend> + <cmd-frontend>",
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
- Se apenas Backend está ativo (sem Frontend), omita o comando de frontend
- Se apenas Frontend está ativo (sem Backend), omita o comando de backend
- Se Docker está ativo mas nenhuma outra camada tem servidor de dev, use `docker compose up` diretamente, sem perguntar
- Se o usuário escolher "Registrar ambos", registre no `CLAUDE.md` com comentários:
  ```
  docker compose up          # ambiente completo (container)
  # ou separadamente:
  <cmd-backend>              # backend
  <cmd-frontend>             # frontend
  ```

---

## Exemplo de Composição Final (PHP + React + Docker)

```
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
