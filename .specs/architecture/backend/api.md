# API — Backend

> Guia de boas práticas para controllers, rotas e validação.
> Complementa o [`spec.md`](./spec.md) deste mesmo diretório.

---

## Regras Gerais [PADRÃO]

- **Controllers:** magros — recebem request, validam, delegam para service, retornam response
- **Validação:** acontece antes da lógica de negócio — nunca dentro do service
- **Rotas:** uma rota por ação — proibido endpoints genéricos que fazem coisas diferentes por query param

> Para regras de estrutura de resposta, status HTTP e paginação, veja "API Design" no [`spec.md`](./spec.md).

---

## Exemplo: Controller (Node.js/TypeScript — Fastify)

```ts
// controllers/user.controller.ts

import type { FastifyInstance } from 'fastify';
import { UserService } from '@/services/user.service';
import { createUserSchema, updateUserSchema } from '@/validators/user.validator';

export const userController = (app: FastifyInstance, userService: UserService) => {
  app.get('/api/v1/users/:id', async (request, reply) => {
    const { id } = request.params as { id: string };
    const user = await userService.getById(id);
    return reply.status(200).send({ data: user });
  });

  app.post('/api/v1/users', async (request, reply) => {
    const data = createUserSchema.parse(request.body);
    const user = await userService.create(data);
    return reply.status(201).send({ data: user });
  });

  app.patch('/api/v1/users/:id', async (request, reply) => {
    const { id } = request.params as { id: string };
    const data = updateUserSchema.parse(request.body);
    const user = await userService.update(id, data);
    return reply.status(200).send({ data: user });
  });

  app.delete('/api/v1/users/:id', async (request, reply) => {
    const { id } = request.params as { id: string };
    await userService.delete(id);
    return reply.status(204).send();
  });
};
```

---

## Exemplo: Controller (PHP/Laravel)

```php
// app/Http/Controllers/UserController.php

<?php

declare(strict_types=1);

namespace App\Http\Controllers;

use App\Http\Requests\CreateUserRequest;
use App\Http\Requests\UpdateUserRequest;
use App\Services\UserService;
use App\DTOs\CreateUserDTO;
use Illuminate\Http\JsonResponse;

class UserController extends Controller
{
    public function __construct(
        private readonly UserService $userService,
    ) {}

    public function show(string $id): JsonResponse
    {
        $user = $this->userService->getById($id);
        return response()->json(['data' => $user]);
    }

    public function store(CreateUserRequest $request): JsonResponse
    {
        $user = $this->userService->create(
            CreateUserDTO::from($request->validated())
        );
        return response()->json(['data' => $user], 201);
    }

    public function update(UpdateUserRequest $request, string $id): JsonResponse
    {
        $user = $this->userService->update($id, $request->validated());
        return response()->json(['data' => $user]);
    }

    public function destroy(string $id): JsonResponse
    {
        $this->userService->delete($id);
        return response()->json(null, 204);
    }
}
```

---

## Exemplo: Validação (Node.js/TypeScript — Zod)

```ts
// validators/user.validator.ts

import { z } from 'zod';

export const createUserSchema = z.object({
  name: z.string().min(2).max(100),
  email: z.string().email(),
  password: z.string().min(8).max(128),
});

export const updateUserSchema = z.object({
  name: z.string().min(2).max(100).optional(),
  email: z.string().email().optional(),
  password: z.string().min(8).max(128).optional(),
});

export type CreateUserDTO = z.infer<typeof createUserSchema>;
export type UpdateUserDTO = z.infer<typeof updateUserSchema>;
```

---

## Exemplo: Validação (PHP/Laravel — FormRequest)

```php
// app/Http/Requests/CreateUserRequest.php

<?php

declare(strict_types=1);

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class CreateUserRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'name' => ['required', 'string', 'min:2', 'max:100'],
            'email' => ['required', 'email', 'unique:users,email'],
            'password' => ['required', 'string', 'min:8', 'max:128', 'confirmed'],
        ];
    }
}
```

---

## Exemplo: Middleware de autenticação (Node.js/TypeScript)

```ts
// middlewares/auth.middleware.ts

import type { FastifyRequest, FastifyReply } from 'fastify';
import { verifyToken } from '@/utils/jwt';
import { UnauthorizedError } from '@/errors';

export const authMiddleware = async (request: FastifyRequest, reply: FastifyReply) => {
  const header = request.headers.authorization;

  if (!header?.startsWith('Bearer ')) {
    throw new UnauthorizedError('Missing or invalid authorization header');
  }

  const token = header.slice(7);

  try {
    const payload = await verifyToken(token);
    request.user = payload;
  } catch {
    throw new UnauthorizedError('Invalid or expired token');
  }
};
```

---

## Exemplo: Error handler global (Node.js/TypeScript)

```ts
// middlewares/error-handler.ts

import type { FastifyError, FastifyReply, FastifyRequest } from 'fastify';
import { AppError } from '@/errors';

export const errorHandler = (error: FastifyError, request: FastifyRequest, reply: FastifyReply) => {
  if (error instanceof AppError) {
    return reply.status(error.statusCode).send({
      error: { message: error.message, code: error.code },
    });
  }

  // Erro de validação (Zod)
  if (error.validation) {
    return reply.status(422).send({
      error: { message: 'Validation failed', code: 'VALIDATION_ERROR', details: error.validation },
    });
  }

  // Erro inesperado — log completo, resposta genérica
  request.log.error(error);
  return reply.status(500).send({
    error: { message: 'Internal server error', code: 'INTERNAL_ERROR' },
  });
};
```

---

## Exemplo: Error handler global (PHP/Laravel)

```php
// app/Exceptions/Handler.php (ou bootstrap/app.php em Laravel 11+)

<?php

declare(strict_types=1);

namespace App\Exceptions;

use Illuminate\Foundation\Exceptions\Handler as ExceptionHandler;
use Illuminate\Http\JsonResponse;
use Illuminate\Validation\ValidationException;
use Throwable;

class Handler extends ExceptionHandler
{
    public function render($request, Throwable $exception): JsonResponse
    {
        if ($exception instanceof AppException) {
            return response()->json($exception->toArray(), $exception->statusCode);
        }

        if ($exception instanceof ValidationException) {
            return response()->json([
                'error' => [
                    'message' => 'Validation failed',
                    'code' => 'VALIDATION_ERROR',
                    'details' => $exception->errors(),
                ],
            ], 422);
        }

        report($exception);

        return response()->json([
            'error' => [
                'message' => 'Internal server error',
                'code' => 'INTERNAL_ERROR',
            ],
        ], 500);
    }
}
```

---

## Exemplo: Paginação padronizada

```ts
// Node.js — resposta de listagem paginada
{
  "data": [...],
  "meta": {
    "page": 1,
    "per_page": 20,
    "total": 147,
    "last_page": 8
  }
}
```

```ts
// utils/pagination.ts

interface PaginationParams {
  page: number;
  perPage: number;
}

interface PaginatedResponse<T> {
  data: T[];
  meta: {
    page: number;
    per_page: number;
    total: number;
    last_page: number;
  };
}

export const paginate = <T>(items: T[], total: number, params: PaginationParams): PaginatedResponse<T> => ({
  data: items,
  meta: {
    page: params.page,
    per_page: params.perPage,
    total,
    last_page: Math.ceil(total / params.perPage),
  },
});
```

---

## Anti-patterns (o que evitar)

```ts
// ❌ Validação dentro do service
export class UserService {
  create = async (data: unknown) => {
    if (!data.email || !data.email.includes('@')) {
      throw new Error('Invalid email');
    }
    // ...
  };
}

// ✅ Validação antes do service (no controller ou middleware)
app.post('/users', async (req, reply) => {
  const data = createUserSchema.parse(req.body);  // valida aqui
  const user = await userService.create(data);     // service recebe dado limpo
  reply.status(201).send({ data: user });
});
```

```ts
// ❌ Respostas inconsistentes
app.get('/users/:id', async (req, reply) => {
  return reply.send(user);           // sem envelope
});
app.get('/users', async (req, reply) => {
  return reply.send({ users: [] });  // envelope diferente
});

// ✅ Estrutura sempre igual
app.get('/users/:id', async (req, reply) => {
  return reply.send({ data: user });
});
app.get('/users', async (req, reply) => {
  return reply.send({ data: users, meta: { ... } });
});
```

```php
// ❌ Endpoint genérico com ações por query param
Route::post('/users/action', function (Request $request) {
    match ($request->input('action')) {
        'create' => ...,
        'delete' => ...,
        'ban' => ...,
    };
});

// ✅ Uma rota por ação
Route::post('/users', [UserController::class, 'store']);
Route::delete('/users/{id}', [UserController::class, 'destroy']);
Route::post('/users/{id}/ban', [UserController::class, 'ban']);
```
