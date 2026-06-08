# Monitoramento — DevOps

> Guia de boas práticas para observabilidade, logging, métricas, health checks e alerting.
> Complementa o [`spec.md`](./spec.md) deste mesmo diretório.

---

## Regras Gerais [PADRÃO]

- **Structured logging:** obrigatório — logs em formato JSON com campos padronizados
- **Health endpoints:** toda aplicação em produção expõe `/healthz` (liveness) e `/readyz` (readiness)
- **Alertas:** configurados antes do primeiro deploy em produção — proibido deploy sem alertas mínimos
- **Dados sensíveis:** proibido logar senhas, tokens, CPF, cartões ou qualquer dado pessoal identificável
- **Retenção:** logs e métricas com política de retenção definida — proibido acumular indefinidamente
- **Correlação:** requests rastreáveis com `traceId` ou `requestId` propagado entre serviços

> Para regras sobre o que logar em erros de aplicação, veja "Tratamento de Erros" no [`spec.md`](./spec.md) do backend.
> Para health checks no container (Docker), veja [`containers.md`](./containers.md).

---

## Exemplo: Health check endpoint — liveness + readiness (Node.js/TypeScript — Fastify)

```ts
// routes/health.ts

import type { FastifyInstance } from 'fastify';
import { db } from '@/lib/database';
import { redis } from '@/lib/redis';

export const healthRoutes = (app: FastifyInstance) => {
  app.get('/healthz', async (_request, reply) => {
    return reply.status(200).send({ status: 'ok' });
  });

  app.get('/readyz', async (_request, reply) => {
    const checks: Record<string, string> = {};

    try {
      await db.raw('SELECT 1');
      checks.database = 'ok';
    } catch {
      checks.database = 'fail';
    }

    try {
      await redis.ping();
      checks.redis = 'ok';
    } catch {
      checks.redis = 'fail';
    }

    const healthy = Object.values(checks).every((v) => v === 'ok');

    return reply.status(healthy ? 200 : 503).send({
      status: healthy ? 'ready' : 'not_ready',
      checks,
    });
  });
};
```

---

## Exemplo: Health check endpoint — liveness + readiness (PHP/Laravel)

```php
// routes/api.php

<?php

use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Redis;
use Illuminate\Support\Facades\Route;

Route::get('/healthz', function () {
    return response()->json(['status' => 'ok']);
});

Route::get('/readyz', function () {
    $checks = [];

    try {
        DB::select('SELECT 1');
        $checks['database'] = 'ok';
    } catch (\Throwable) {
        $checks['database'] = 'fail';
    }

    try {
        Redis::ping();
        $checks['redis'] = 'ok';
    } catch (\Throwable) {
        $checks['redis'] = 'fail';
    }

    $healthy = !in_array('fail', $checks, true);

    return response()->json([
        'status' => $healthy ? 'ready' : 'not_ready',
        'checks' => $checks,
    ], $healthy ? 200 : 503);
});
```

---

## Exemplo: Structured logging

Logs devem ser emitidos em JSON com campos padronizados. Ferramentas de agregação (ELK, Grafana Loki, Datadog, etc.) conseguem indexar e buscar por qualquer campo.

### Formato padrão de log

```json
{
  "timestamp": "2025-01-15T14:30:00.123Z",
  "level": "info",
  "service": "api",
  "traceId": "abc-123-def-456",
  "message": "User created",
  "data": {
    "userId": "usr_789",
    "email": "user@example.com"
  }
}
```

### Campos obrigatórios

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `timestamp` | ISO 8601 | Momento exato do evento |
| `level` | string | `debug`, `info`, `warn`, `error` |
| `service` | string | Nome do serviço que emitiu o log |
| `message` | string | Descrição do evento |

### Campos recomendados

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `traceId` | string | Identificador para rastrear o request entre serviços |
| `userId` | string | Usuário autenticado (quando aplicável) |
| `data` | object | Dados adicionais relevantes (sem dados sensíveis) |
| `error` | object | Stack trace e detalhes do erro (apenas em nível `error`) |
| `duration` | number | Tempo de execução em milissegundos |

### Implementação (Node.js — Pino)

```ts
// lib/logger.ts

import pino from 'pino';
import { env } from '@/config/env';

export const logger = pino({
  level: env.LOG_LEVEL,
  formatters: {
    level: (label) => ({ level: label }),
  },
  base: {
    service: 'api',
  },
  timestamp: pino.stdTimeFunctions.isoTime,
  redact: ['req.headers.authorization', 'data.password', 'data.token'],
});
```

### Implementação (PHP/Laravel — Monolog JSON)

```php
// config/logging.php (canal customizado)

'channels' => [
    'json' => [
        'driver' => 'monolog',
        'handler' => StreamHandler::class,
        'handler_with' => [
            'stream' => 'php://stdout',
        ],
        'formatter' => JsonFormatter::class,
    ],
],
```

---

## Exemplo: Métricas de aplicação — RED method

O método RED define as três métricas essenciais para monitorar qualquer serviço:

| Métrica | O que mede | Exemplo |
|---------|-----------|---------|
| **Rate** | Requests por segundo | `http_requests_total` |
| **Errors** | Taxa de erros (%) | `http_errors_total / http_requests_total` |
| **Duration** | Latência por request | `http_request_duration_seconds` (p50, p95, p99) |

### Implementação agnóstica

Independente da ferramenta (Prometheus, Datadog, New Relic, CloudWatch), toda aplicação deve expor ao menos:

```
# Contadores
requests_total{method, path, status}     — total de requests
errors_total{method, path, status}       — total de erros (4xx + 5xx)

# Histograma
request_duration_seconds{method, path}   — latência por rota
```

### Dashboard mínimo

Um dashboard de aplicação deve responder a quatro perguntas:

1. **Quantos requests estamos recebendo?** — Rate (requests/segundo)
2. **Quantos estão falhando?** — Error rate (% de 5xx)
3. **Quanto tempo estão levando?** — Latência (p50, p95, p99)
4. **Os serviços dependentes estão saudáveis?** — Health checks (database, cache, filas)

---

## Exemplo: Alerting rules

Alertas devem ser acionáveis — cada alerta precisa de um responsável e um runbook (ou instruções de investigação).

### Thresholds mínimos recomendados

| Alerta | Condição | Severidade |
|--------|---------|------------|
| Error rate alto | > 5% de 5xx por 5 minutos | **critical** |
| Latência alta | p95 > 2s por 5 minutos | **warning** |
| Latência muito alta | p95 > 5s por 2 minutos | **critical** |
| Health check falhando | `/readyz` retornando 503 por 2 minutos | **critical** |
| Disco cheio | > 85% de uso | **warning** |
| Disco crítico | > 95% de uso | **critical** |
| CPU sustentada | > 80% por 10 minutos | **warning** |
| Memória alta | > 90% de uso por 5 minutos | **warning** |

### Anatomia de um alerta bem definido

```yaml
# Formato conceitual — adaptar para a ferramenta usada (Grafana, PagerDuty, Datadog, etc.)

name: high-error-rate
description: Taxa de erros HTTP 5xx acima do threshold
condition: error_rate_5xx > 5% for 5m
severity: critical
notify:
  - channel: "#alerts-critical"
  - oncall: backend-team
runbook: https://wiki.example.com/runbooks/high-error-rate
```

> Todo alerta `critical` deve ter runbook. Alertas sem runbook geram fadiga e são ignorados com o tempo.

---

## Exemplo: Error tracking

Integrar um serviço de error tracking (Sentry, Bugsnag, Rollbar, etc.) com padrão agnóstico de vendor:

```ts
// lib/error-tracker.ts

interface ErrorTracker {
  captureException(error: Error, context?: Record<string, unknown>): void;
  setUser(user: { id: string; email: string }): void;
}

export const createErrorTracker = (): ErrorTracker => {
  // Adaptar para o serviço escolhido (Sentry, Bugsnag, etc.)
  return {
    captureException(error, context) {
      // Sentry: Sentry.captureException(error, { extra: context })
      // Bugsnag: bugsnag.notify(error, event => event.addMetadata('context', context))
    },
    setUser(user) {
      // Sentry: Sentry.setUser(user)
      // Bugsnag: bugsnag.setUser(user.id, user.email)
    },
  };
};
```

> A abstração evita vendor lock-in. O serviço de error tracking pode mudar sem afetar o código da aplicação.

---

## Anti-patterns (o que evitar)

```ts
// ❌ Logs sem estrutura — impossível buscar e filtrar
console.log('User created: ' + userId);
console.log('Error:', error.message);

// ✅ Structured logging com campos indexáveis
logger.info({ userId, action: 'user_created' }, 'User created');
logger.error({ err: error, userId, route: '/users' }, 'Failed to create user');
```

```ts
// ❌ Health check que sempre retorna 200 — não verifica dependências
app.get('/healthz', (req, res) => {
  res.json({ status: 'ok' });
});
// Se o banco cair, o health check ainda diz "ok"

// ✅ Readiness check que verifica dependências reais
app.get('/readyz', async (req, res) => {
  const dbOk = await checkDatabase();
  const redisOk = await checkRedis();
  const healthy = dbOk && redisOk;
  res.status(healthy ? 200 : 503).json({
    status: healthy ? 'ready' : 'not_ready',
    checks: { database: dbOk ? 'ok' : 'fail', redis: redisOk ? 'ok' : 'fail' },
  });
});
```

```ts
// ❌ Logar dados sensíveis
logger.info({ password: user.password, token: authToken }, 'User login');

// ✅ Redact de campos sensíveis — configurado globalmente no logger
// Pino: redact: ['password', 'token', 'req.headers.authorization']
logger.info({ userId: user.id, action: 'login' }, 'User login');
```

```yaml
# ❌ Alertas sem runbook — ninguém sabe o que fazer quando dispara
name: high-error-rate
condition: error_rate > 5%
severity: critical
# ...e agora?

# ✅ Alerta com runbook e responsável claro
name: high-error-rate
condition: error_rate > 5%
severity: critical
runbook: https://wiki.example.com/runbooks/high-error-rate
notify: "#alerts-critical"
```

```ts
// ❌ Error tracking sem contexto
errorTracker.captureException(error);
// Recebe o erro mas não sabe qual usuário, qual rota, qual request

// ✅ Error tracking com contexto suficiente para reprodução
errorTracker.captureException(error, {
  userId: request.user?.id,
  route: request.url,
  method: request.method,
  body: sanitize(request.body),
});
```
