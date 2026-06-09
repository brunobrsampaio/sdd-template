# Testes — Backend

> Guia de boas práticas para testes em projetos backend (Node.js e PHP).
> Complementa o [`spec.md`](./spec.md) deste mesmo diretório.

---

## Regras Gerais [PADRÃO]

- **Abordagem:** TDD — testes escritos antes do código de produção. Quando inviável, o teste entra no mesmo commit que a implementação — nunca em PR separado.
- **Cobertura mínima:** 80% de branches na camada de lógica (serviços)
- **Unitários:** isolam lógica de negócio de I/O (banco, HTTP, filesystem)
- **Integração:** rodam contra banco real em container isolado
- **Independência:** cada teste é independente — proibido depender de ordem de execução
- **Limpeza:** setup e teardown limpam o estado entre testes

---

## Estrutura de Testes

```
tests/ (ou __tests__/)
├── unit/
│   └── services/
│       └── user.service.test.ts
├── integration/
│   └── routes/
│       └── users.test.ts
└── helpers/
    └── factories.ts         ← Factories para gerar dados de teste
```

> Em PHP/Laravel, a convenção é `tests/Unit/` e `tests/Feature/`. Siga a convenção do framework.

---

## Exemplo: Teste unitário de service (Node.js/TypeScript)

```ts
// tests/unit/services/user.service.test.ts

import { UserService } from '@/services/user.service';
import type { UserRepository } from '@/repositories/user.repository';

const mockRepository: jest.Mocked<UserRepository> = {
  findById: jest.fn(),
  findByEmail: jest.fn(),
  create: jest.fn(),
  update: jest.fn(),
  delete: jest.fn(),
};

const userService = new UserService(mockRepository);

describe('UserService', () => {
  beforeEach(() => jest.clearAllMocks());

  describe('getById', () => {
    it('returns user when found', async () => {
      mockRepository.findById.mockResolvedValue({
        id: '123',
        name: 'João',
        email: 'joao@example.com',
      });

      const user = await userService.getById('123');

      expect(user).toEqual({ id: '123', name: 'João', email: 'joao@example.com' });
      expect(mockRepository.findById).toHaveBeenCalledWith('123');
    });

    it('throws UserNotFoundError when not found', async () => {
      mockRepository.findById.mockResolvedValue(null);

      await expect(userService.getById('999')).rejects.toThrow('User not found');
    });
  });

  describe('create', () => {
    it('creates user with hashed password', async () => {
      mockRepository.findByEmail.mockResolvedValue(null);
      mockRepository.create.mockResolvedValue({ id: '1', name: 'Maria', email: 'maria@example.com' });

      const user = await userService.create({
        name: 'Maria',
        email: 'maria@example.com',
        password: 'senha123',
      });

      expect(user.id).toBe('1');
      expect(mockRepository.create).toHaveBeenCalledWith(
        expect.objectContaining({
          name: 'Maria',
          email: 'maria@example.com',
          password: expect.not.stringMatching('senha123'),
        })
      );
    });

    it('throws when email already exists', async () => {
      mockRepository.findByEmail.mockResolvedValue({ id: '1', email: 'maria@example.com' });

      await expect(
        userService.create({ name: 'Maria', email: 'maria@example.com', password: '123' })
      ).rejects.toThrow('Email already in use');
    });
  });
});
```

---

## Exemplo: Teste unitário de service (PHP/Laravel)

```php
// tests/Unit/Services/UserServiceTest.php

<?php

declare(strict_types=1);

namespace Tests\Unit\Services;

use App\Services\UserService;
use App\Repositories\UserRepository;
use App\Exceptions\UserNotFoundException;
use PHPUnit\Framework\TestCase;
use Mockery;

class UserServiceTest extends TestCase
{
    private UserService $service;
    private UserRepository $repository;

    protected function setUp(): void
    {
        $this->repository = Mockery::mock(UserRepository::class);
        $this->service = new UserService($this->repository);
    }

    protected function tearDown(): void
    {
        Mockery::close();
    }

    public function test_returns_user_when_found(): void
    {
        $this->repository
            ->shouldReceive('findById')
            ->with('123')
            ->andReturn(['id' => '123', 'name' => 'João', 'email' => 'joao@example.com']);

        $user = $this->service->getById('123');

        $this->assertEquals('João', $user['name']);
    }

    public function test_throws_when_user_not_found(): void
    {
        $this->repository
            ->shouldReceive('findById')
            ->with('999')
            ->andReturn(null);

        $this->expectException(UserNotFoundException::class);

        $this->service->getById('999');
    }
}
```

---

## Exemplo: Teste de integração de rota (Node.js/TypeScript)

```ts
// tests/integration/routes/users.test.ts

import { describe, it, expect, beforeAll, afterAll, beforeEach } from 'vitest';
import { buildApp } from '@/app';
import { db } from '@/database';
import { userFactory } from '../helpers/factories';

let app: ReturnType<typeof buildApp>;

beforeAll(async () => {
  app = buildApp();
  await app.ready();
});

afterAll(async () => {
  await app.close();
});

beforeEach(async () => {
  await db.delete(users);
});

describe('GET /api/v1/users/:id', () => {
  it('returns 200 with user data', async () => {
    const user = await userFactory.create();

    const response = await app.inject({
      method: 'GET',
      url: `/api/v1/users/${user.id}`,
      headers: { authorization: `Bearer ${validToken}` },
    });

    expect(response.statusCode).toBe(200);
    expect(response.json()).toEqual({
      data: expect.objectContaining({
        id: user.id,
        name: user.name,
        email: user.email,
      }),
    });
  });

  it('returns 404 when user not found', async () => {
    const response = await app.inject({
      method: 'GET',
      url: '/api/v1/users/nonexistent-id',
      headers: { authorization: `Bearer ${validToken}` },
    });

    expect(response.statusCode).toBe(404);
    expect(response.json()).toEqual({
      error: { message: 'User not found', code: 'USER_NOT_FOUND' },
    });
  });

  it('returns 401 without auth header', async () => {
    const response = await app.inject({
      method: 'GET',
      url: '/api/v1/users/123',
    });

    expect(response.statusCode).toBe(401);
  });
});

describe('POST /api/v1/users', () => {
  it('returns 201 on successful creation', async () => {
    const response = await app.inject({
      method: 'POST',
      url: '/api/v1/users',
      headers: { authorization: `Bearer ${adminToken}` },
      payload: {
        name: 'Novo User',
        email: 'novo@example.com',
        password: 'senha123',
      },
    });

    expect(response.statusCode).toBe(201);
    expect(response.json().data).toHaveProperty('id');
  });

  it('returns 422 with validation errors', async () => {
    const response = await app.inject({
      method: 'POST',
      url: '/api/v1/users',
      headers: { authorization: `Bearer ${adminToken}` },
      payload: { name: '', email: 'invalido' },
    });

    expect(response.statusCode).toBe(422);
    expect(response.json().error).toHaveProperty('message');
  });
});
```

---

## Exemplo: Teste de integração de rota (PHP/Laravel)

```php
// tests/Feature/Routes/UsersTest.php

<?php

declare(strict_types=1);

namespace Tests\Feature\Routes;

use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class UsersTest extends TestCase
{
    use RefreshDatabase;

    public function test_returns_user_when_found(): void
    {
        $user = User::factory()->create();

        $response = $this->actingAs($user)->getJson("/api/v1/users/{$user->id}");

        $response->assertOk()
            ->assertJsonStructure(['data' => ['id', 'name', 'email']]);
    }

    public function test_returns_404_when_not_found(): void
    {
        $user = User::factory()->create();

        $response = $this->actingAs($user)->getJson('/api/v1/users/nonexistent-id');

        $response->assertNotFound()
            ->assertJson(['error' => ['code' => 'USER_NOT_FOUND']]);
    }

    public function test_returns_401_without_auth(): void
    {
        $response = $this->getJson('/api/v1/users/123');

        $response->assertUnauthorized();
    }

    public function test_creates_user_successfully(): void
    {
        $admin = User::factory()->admin()->create();

        $response = $this->actingAs($admin)->postJson('/api/v1/users', [
            'name' => 'Novo User',
            'email' => 'novo@example.com',
            'password' => 'senha123',
            'password_confirmation' => 'senha123',
        ]);

        $response->assertCreated()
            ->assertJsonStructure(['data' => ['id']]);

        $this->assertDatabaseHas('users', ['email' => 'novo@example.com']);
    }

    public function test_returns_422_with_validation_errors(): void
    {
        $admin = User::factory()->admin()->create();

        $response = $this->actingAs($admin)->postJson('/api/v1/users', [
            'name' => '',
            'email' => 'invalido',
        ]);

        $response->assertUnprocessable()
            ->assertJsonValidationErrors(['name', 'email', 'password']);
    }
}
```

---

## Exemplo: Factory para testes

```ts
// tests/helpers/factories.ts (Node.js)

import { db } from '@/database';
import { users } from '@/models/schema';
import { randomUUID } from 'crypto';

export const userFactory = {
  build: (overrides = {}) => ({
    id: randomUUID(),
    name: 'Test User',
    email: `user-${randomUUID().slice(0, 8)}@test.com`,
    password: 'hashed_password',
    createdAt: new Date(),
    ...overrides,
  }),

  create: async (overrides = {}) => {
    const data = userFactory.build(overrides);
    const [user] = await db.insert(users).values(data).returning();
    return user;
  },
};
```

---

## Anti-patterns (o que evitar)

```ts
// ❌ Teste acoplado ao banco sem isolamento
it('creates user', async () => {
  await userService.create({ name: 'João', email: 'joao@test.com', password: '123' });
  // não limpa o banco — próximo teste pode falhar
});

// ✅ Isolamento entre testes
beforeEach(async () => {
  await db.delete(users);
});
```

```ts
// ❌ Testar detalhes de implementação
it('calls bcrypt.hash with 12 rounds', async () => {
  await userService.create(data);
  expect(bcrypt.hash).toHaveBeenCalledWith('senha', 12);
});

// ✅ Testar comportamento observável
it('stores hashed password (not plaintext)', async () => {
  await userService.create({ ...data, password: 'senha123' });
  const stored = await repository.findByEmail(data.email);
  expect(stored.password).not.toBe('senha123');
});
```

```ts
// ❌ Mock de tudo — teste não prova nada
it('returns user', async () => {
  mockService.getById.mockResolvedValue(mockUser);
  const result = await controller.getUser('123');
  expect(result).toBe(mockUser);  // só prova que o mock funciona
});

// ✅ Teste de integração valida o fluxo completo
it('returns user via HTTP', async () => {
  const user = await userFactory.create();
  const response = await app.inject({ method: 'GET', url: `/api/v1/users/${user.id}` });
  expect(response.statusCode).toBe(200);
  expect(response.json().data.name).toBe(user.name);
});
```
