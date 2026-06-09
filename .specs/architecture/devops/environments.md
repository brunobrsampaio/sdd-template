# Ambientes e Variáveis — DevOps

> Guia de boas práticas para gestão de ambientes, variáveis de ambiente, secrets e feature flags.
> Complementa o [`spec.md`](./spec.md) deste mesmo diretório.

---

## Regras Gerais [PADRÃO]

As regras de variáveis e secrets (isolamento por ambiente, nomenclatura `SCREAMING_SNAKE_CASE` prefixada, `.env.example` documentado, proibição de commitar `.env` real e armazenamento de secrets no cofre da plataforma) estão em "Ambientes", "Variáveis de Ambiente" e "Secrets e Segurança" no [`spec.md`](./spec.md). Este guia adiciona:

- **Nomenclatura de ambiente:** o ambiente `local` corresponde a `NODE_ENV=development` (Node) / `APP_ENV=local` (PHP) — `staging` e `production` mantêm o mesmo nome em ambos
- **Validação:** a aplicação valida variáveis obrigatórias no boot — falha rápido se faltar alguma

> Para regras de nomenclatura e formato do `.env.example`, veja "Variáveis de Ambiente" no [`spec.md`](./spec.md).
> Para configuração de secrets no pipeline, veja [`ci.md`](./ci.md).

---

## Exemplo: .env.example

```bash
# .env.example
# Copie para .env e preencha os valores

# ── Aplicação ──────────────────────────────────────────
NODE_ENV=local                   # local | staging | production
APP_PORT=3000                    # Porta da aplicação
APP_URL=http://localhost:3000    # URL base da aplicação

# ── Banco de Dados ────────────────────────────────────
DATABASE_URL=                    # URL de conexão (ex: postgresql://user:pass@host:5432/db)
DATABASE_POOL_SIZE=10            # Número máximo de conexões no pool

# ── Cache ──────────────────────────────────────────────
REDIS_URL=                       # URL de conexão Redis (ex: redis://host:6379)

# ── Autenticação ──────────────────────────────────────
JWT_SECRET=                      # Secret para assinatura de tokens JWT
JWT_EXPIRES_IN=15m               # Tempo de expiração do access token
REFRESH_TOKEN_EXPIRES_IN=7d      # Tempo de expiração do refresh token

# ── Serviços Externos ─────────────────────────────────
STRIPE_SECRET_KEY=               # Chave secreta da API do Stripe
STRIPE_WEBHOOK_SECRET=           # Secret do webhook do Stripe
SENDGRID_API_KEY=                # Chave da API do SendGrid

# ── Observabilidade ───────────────────────────────────
SENTRY_DSN=                      # DSN do Sentry para error tracking
LOG_LEVEL=debug                  # debug | info | warn | error
```

---

## Exemplo: Validação de env vars no boot (Node.js/TypeScript — Zod)

```ts
// config/env.ts

import { z } from 'zod';

const envSchema = z.object({
  NODE_ENV: z.enum(['local', 'staging', 'production']),
  APP_PORT: z.coerce.number().default(3000),
  APP_URL: z.string().url(),

  DATABASE_URL: z.string().min(1),
  DATABASE_POOL_SIZE: z.coerce.number().default(10),

  REDIS_URL: z.string().min(1),

  JWT_SECRET: z.string().min(32),
  JWT_EXPIRES_IN: z.string().default('15m'),

  SENTRY_DSN: z.string().url().optional(),
  LOG_LEVEL: z.enum(['debug', 'info', 'warn', 'error']).default('info'),
});

export const env = envSchema.parse(process.env);
```

> A aplicação falha imediatamente no boot se alguma variável obrigatória estiver ausente ou com formato inválido — sem mensagens genéricas. O Zod indica exatamente qual campo falhou.

---

## Exemplo: Validação de env vars no boot (PHP/Laravel)

```php
// app/Providers/AppServiceProvider.php

<?php

declare(strict_types=1);

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use RuntimeException;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        $this->validateEnvironment();
    }

    private function validateEnvironment(): void
    {
        $required = [
            'APP_KEY',
            'DB_CONNECTION',
            'DB_HOST',
            'DB_DATABASE',
            'JWT_SECRET',
        ];

        $missing = array_filter(
            $required,
            fn (string $key): bool => empty(env($key))
        );

        if (!empty($missing)) {
            throw new RuntimeException(
                'Missing required environment variables: ' . implode(', ', $missing)
            );
        }
    }
}
```

> Em produção, `env()` deve ser chamado apenas durante o boot. Para leitura em runtime, espelhe os valores em `config/*.php` e use `config('chave')` — assim `php artisan config:cache` continua funcionando (o cache de configuração derruba a leitura direta de `env()` em runtime).

---

## Exemplo: Secrets no CI (GitHub Actions)

Secrets são configurados em **Settings → Secrets and variables → Actions**.

```yaml
# .github/workflows/deploy.yml

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4
      - run: deploy --token "$API_TOKEN" --url "$DEPLOY_URL"
        env:
          API_TOKEN: ${{ secrets.API_TOKEN }}
          DEPLOY_URL: ${{ secrets.DEPLOY_URL }}
```

- **Repository secrets:** disponíveis em todos os workflows do repositório
- **Environment secrets:** disponíveis apenas em jobs vinculados ao environment — preferir para produção

---

## Exemplo: Secrets no CI (GitLab CI)

Secrets são configurados em **Settings → CI/CD → Variables**.

```yaml
# .gitlab-ci.yml

deploy-production:
  stage: deploy
  script:
    - deploy --token "$API_TOKEN" --url "$DEPLOY_URL"
  environment:
    name: production
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
```

- **Protected:** variável disponível apenas em branches/tags protegidas
- **Masked:** valor ocultado dos logs do pipeline — obrigatório para tokens e senhas

---

## Exemplo: Secrets no CI (Bitbucket Pipelines)

Secrets são configurados em **Repository settings → Pipelines → Repository variables**.

```yaml
# bitbucket-pipelines.yml

pipelines:
  branches:
    main:
      - step:
          name: Deploy Production
          deployment: production
          script:
            - deploy --token "$API_TOKEN" --url "$DEPLOY_URL"
```

- **Secured:** valor criptografado e ocultado dos logs — obrigatório para dados sensíveis
- **Deployment variables:** vinculadas a um environment específico — preferir para produção

---

## Exemplo: Promoção entre ambientes

O artefato (imagem Docker ou bundle) é construído uma única vez e promovido entre ambientes. Proibido rebuild entre staging e production.

```
build → imagem:abc123
  ├── deploy staging  → imagem:abc123 + env staging
  └── deploy production → imagem:abc123 + env production (mesma imagem)
```

```yaml
# GitHub Actions — mesma imagem para staging e production
jobs:
  build:
    steps:
      - run: docker build -t app:${{ github.sha }} .
      - run: docker push registry.example.com/app:${{ github.sha }}

  deploy-staging:
    needs: build
    environment: staging
    steps:
      - run: deploy registry.example.com/app:${{ github.sha }} --env staging

  deploy-production:
    needs: deploy-staging
    environment: production
    steps:
      - run: deploy registry.example.com/app:${{ github.sha }} --env production
```

> O comportamento da aplicação muda pelas variáveis de ambiente — nunca pelo artefato.

---

## Exemplo: Feature flags por ambiente

Feature flags controlam funcionalidades por ambiente sem necessidade de deploy.

```bash
# .env.example
FEATURE_NEW_CHECKOUT=false       # Habilita o novo fluxo de checkout
FEATURE_DARK_MODE=false          # Habilita dark mode na UI
```

```ts
// config/features.ts

import { env } from './env';

export const features = {
  newCheckout: env.FEATURE_NEW_CHECKOUT === 'true',
  darkMode: env.FEATURE_DARK_MODE === 'true',
} as const;
```

```ts
// Uso na aplicação
import { features } from '@/config/features';

if (features.newCheckout) {
  // novo fluxo
} else {
  // fluxo atual
}
```

> Para sistemas mais complexos (rollout gradual, segmentação por usuário), considere serviços dedicados de feature flags. O pattern com env var é suficiente para flags binárias por ambiente.

---

## Anti-patterns (o que evitar)

```bash
# ❌ .env com valores reais commitado no repositório
DATABASE_URL=postgresql://admin:s3cr3t@prod-db:5432/app
STRIPE_SECRET_KEY=sk_live_abc123

# ✅ Apenas .env.example no repositório — sem valores reais
DATABASE_URL=
STRIPE_SECRET_KEY=
```

```ts
// ❌ Sem validação no boot — erro aparece quando a feature é usada
const jwtSecret = process.env.JWT_SECRET; // pode ser undefined
// ...minutos depois, em outra parte do código:
jwt.sign(payload, jwtSecret); // TypeError: secret must be a string

// ✅ Validação no boot — erro imediato e descritivo
const env = envSchema.parse(process.env);
// ZodError: JWT_SECRET is required
```

```yaml
# ❌ Secrets em plaintext no repositório
env:
  API_TOKEN: "sk_live_abc123"

# ✅ Secrets referenciados do cofre da plataforma
env:
  API_TOKEN: ${{ secrets.API_TOKEN }}
```

```bash
# ❌ Variáveis compartilhadas entre ambientes
# Mesmo DATABASE_URL para staging e production

# ✅ Cada ambiente com suas próprias variáveis
# staging: DATABASE_URL=postgresql://...staging-db.../app_staging
# production: DATABASE_URL=postgresql://...prod-db.../app_production
```

```dockerfile
# ❌ Secrets como build args no Dockerfile
ARG STRIPE_SECRET_KEY
ENV STRIPE_SECRET_KEY=$STRIPE_SECRET_KEY
# Fica gravado na imagem — qualquer pessoa com acesso à imagem vê o secret

# ✅ Secrets via variável de ambiente no runtime
# docker run -e STRIPE_SECRET_KEY=sk_live_... app:latest
```
