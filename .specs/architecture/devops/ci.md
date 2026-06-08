# CI/CD — DevOps

> Guia de boas práticas para pipelines de integração e entrega contínua.
> Complementa o [`spec.md`](./spec.md) deste mesmo diretório.

---

## Regras Gerais [PADRÃO]

- **Pipeline como código:** toda definição de pipeline vive no repositório — proibido configuração manual na UI
- **Fail-fast:** pipeline aborta na primeira falha — proibido continuar stages com erros anteriores
- **Caching:** obrigatório para dependências — proibido reinstalar tudo em cada execução
- **Artefatos:** retenção definida explicitamente — proibido artefatos sem prazo de expiração
- **Secrets:** proibido imprimir, exportar ou logar variáveis sensíveis no pipeline
- **Tags de imagem:** proibido usar `latest` em imagens de CI — use tags específicas com versão

> Para a ordem obrigatória de stages (lint → typecheck → test → build → deploy), veja "CI Pipeline" no [`spec.md`](./spec.md).
> Para regras de armazenamento de secrets, veja [`environments.md`](./environments.md).

---

## Exemplo: Pipeline básico (GitHub Actions)

```yaml
# .github/workflows/ci.yml

name: CI

on:
  pull_request:
    branches: [main, staging]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm run lint

  typecheck:
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm run typecheck

  test:
    runs-on: ubuntu-latest
    needs: typecheck
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm test

  build:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm run build
```

---

## Exemplo: Pipeline básico (GitLab CI)

```yaml
# .gitlab-ci.yml

stages:
  - lint
  - typecheck
  - test
  - build

default:
  image: node:20-alpine
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - node_modules/

before_script:
  - npm ci

lint:
  stage: lint
  script:
    - npm run lint

typecheck:
  stage: typecheck
  script:
    - npm run typecheck

test:
  stage: test
  script:
    - npm test

build:
  stage: build
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 7 days
```

---

## Exemplo: Pipeline básico (Bitbucket Pipelines)

```yaml
# bitbucket-pipelines.yml

image: node:20-alpine

definitions:
  caches:
    npm: ~/.npm

pipelines:
  pull-requests:
    '**':
      - step:
          name: Lint
          caches:
            - npm
          script:
            - npm ci
            - npm run lint
      - step:
          name: Typecheck
          caches:
            - npm
          script:
            - npm ci
            - npm run typecheck
      - step:
          name: Test
          caches:
            - npm
          script:
            - npm ci
            - npm test
      - step:
          name: Build
          caches:
            - npm
          script:
            - npm ci
            - npm run build
```

---

## Exemplo: Deploy com ambientes (GitHub Actions)

```yaml
# .github/workflows/deploy.yml

name: Deploy

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/
          retention-days: 7

  deploy-staging:
    runs-on: ubuntu-latest
    needs: build
    environment: staging
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: dist
      - run: echo "Deploy to staging"
        # Substituir pelo comando real de deploy

  deploy-production:
    runs-on: ubuntu-latest
    needs: deploy-staging
    environment:
      name: production
      # Aprovação manual configurada nas settings do environment
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: dist
      - run: echo "Deploy to production"
```

---

## Exemplo: Deploy com ambientes (GitLab CI)

```yaml
# .gitlab-ci.yml (trecho de deploy)

stages:
  - build
  - deploy

build:
  stage: build
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 7 days

deploy-staging:
  stage: deploy
  script:
    - echo "Deploy to staging"
  environment:
    name: staging
    url: https://staging.example.com
  rules:
    - if: $CI_COMMIT_BRANCH == "main"

deploy-production:
  stage: deploy
  script:
    - echo "Deploy to production"
  environment:
    name: production
    url: https://example.com
  when: manual
  needs:
    - deploy-staging
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
```

---

## Exemplo: Deploy com ambientes (Bitbucket Pipelines)

```yaml
# bitbucket-pipelines.yml (trecho de deploy)

pipelines:
  branches:
    main:
      - step:
          name: Build
          caches:
            - npm
          script:
            - npm ci
            - npm run build
          artifacts:
            - dist/**
      - step:
          name: Deploy Staging
          deployment: staging
          script:
            - echo "Deploy to staging"
      - step:
          name: Deploy Production
          deployment: production
          trigger: manual
          script:
            - echo "Deploy to production"
```

---

## Exemplo: Caching de dependências

Cada plataforma tem sua estratégia de cache. O princípio é o mesmo: cachear o diretório de dependências com uma chave baseada no lockfile.

```yaml
# GitHub Actions
- uses: actions/setup-node@v4
  with:
    node-version: 20
    cache: npm
# Alternativa para PHP
- uses: actions/cache@v4
  with:
    path: vendor
    key: composer-${{ hashFiles('composer.lock') }}
```

```yaml
# GitLab CI
cache:
  key:
    files:
      - package-lock.json  # ou composer.lock
  paths:
    - node_modules/        # ou vendor/
```

```yaml
# Bitbucket Pipelines
definitions:
  caches:
    npm: ~/.npm
    composer: ~/.composer/cache
# Usado no step:
- step:
    caches:
      - npm  # ou composer
```

---

## Exemplo: Workflows reutilizáveis (GitHub Actions)

```yaml
# .github/workflows/reusable-ci.yml

name: Reusable CI

on:
  workflow_call:
    inputs:
      node-version:
        required: false
        type: string
        default: '20'

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
          cache: npm
      - run: npm ci
      - run: npm run lint
      - run: npm run typecheck
      - run: npm test
      - run: npm run build
```

```yaml
# .github/workflows/ci.yml — consumindo o workflow reutilizável

name: CI
on:
  pull_request:
    branches: [main]

jobs:
  ci:
    uses: ./.github/workflows/reusable-ci.yml
    with:
      node-version: '20'
```

---

## Exemplo: Templates reutilizáveis (GitLab CI)

```yaml
# .gitlab/ci/templates.yml

.node-setup:
  image: node:20-alpine
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - node_modules/
  before_script:
    - npm ci
```

```yaml
# .gitlab-ci.yml — consumindo o template

include:
  - local: '.gitlab/ci/templates.yml'

lint:
  extends: .node-setup
  stage: lint
  script:
    - npm run lint

test:
  extends: .node-setup
  stage: test
  script:
    - npm test
```

---

## Exemplo: Definições reutilizáveis (Bitbucket Pipelines)

```yaml
# bitbucket-pipelines.yml

definitions:
  caches:
    npm: ~/.npm
  steps:
    - step: &lint-step
        name: Lint
        caches:
          - npm
        script:
          - npm ci
          - npm run lint
    - step: &test-step
        name: Test
        caches:
          - npm
        script:
          - npm ci
          - npm test

pipelines:
  pull-requests:
    '**':
      - step: *lint-step
      - step: *test-step
```

---

## Anti-patterns (o que evitar)

```yaml
# ❌ Secrets expostos em logs
steps:
  - run: echo "Token is ${{ secrets.API_TOKEN }}"
  - run: env | grep TOKEN

# ✅ Secrets usados apenas em contexto seguro — nunca impressos
steps:
  - run: deploy --token "$API_TOKEN"
    env:
      API_TOKEN: ${{ secrets.API_TOKEN }}
```

```yaml
# ❌ Pipeline sem cache — reinstala tudo a cada execução
steps:
  - run: npm install
  - run: npm test

# ✅ Cache de dependências com chave baseada no lockfile
steps:
  - uses: actions/setup-node@v4
    with:
      node-version: 20
      cache: npm
  - run: npm ci
  - run: npm test
```

```yaml
# ❌ Deploy direto para produção sem gate
deploy:
  stage: deploy
  script:
    - deploy-to-production.sh
  rules:
    - if: $CI_COMMIT_BRANCH == "main"

# ✅ Deploy com aprovação manual para produção
deploy-staging:
  stage: deploy
  script:
    - deploy-to-staging.sh
  environment:
    name: staging

deploy-production:
  stage: deploy
  script:
    - deploy-to-production.sh
  environment:
    name: production
  when: manual
  needs:
    - deploy-staging
```

```yaml
# ❌ Jobs duplicados sem template
lint-api:
  image: node:20-alpine
  before_script: [npm ci]
  script: [npm run lint]

lint-web:
  image: node:20-alpine
  before_script: [npm ci]
  script: [npm run lint]

# ✅ Template reutilizável
.node-lint:
  image: node:20-alpine
  before_script: [npm ci]
  script: [npm run lint]

lint-api:
  extends: .node-lint
  variables:
    WORKING_DIR: api

lint-web:
  extends: .node-lint
  variables:
    WORKING_DIR: web
```
