# Services — Backend

> Guia de boas práticas para implementação de lógica de negócio.
> Complementa o [`spec.md`](./spec.md) deste mesmo diretório.

---

## Regras Gerais [PADRÃO]

- **Responsabilidade:** services contêm toda a lógica de negócio — controllers apenas delegam
- **Isolamento:** services são agnósticos de HTTP — não conhecem Request/Response
- **Injeção de dependência:** dependências via construtor (DI) — facilita testes e desacoplamento
- **Granularidade:** um service por domínio/entidade (`UserService`, `OrderService`)
- **Tipagem:** retorno explícito — nunca `any` ou `mixed`

> Para regras de tratamento de erros e tipagem, veja "Tratamento de Erros" no [`spec.md`](./spec.md).

---

## Exemplo: Service (Node.js/TypeScript)

```ts
// services/user.service.ts

import type { UserRepository } from '@/repositories/user.repository';
import type { CreateUserDTO, UserResponse } from '@/types/user';
import { hashPassword } from '@/utils/crypto';
import { UserNotFoundError, EmailAlreadyExistsError } from '@/errors';

export class UserService {
  constructor(private readonly userRepository: UserRepository) {}

  getById = async (id: string): Promise<UserResponse> => {
    const user = await this.userRepository.findById(id);
    if (!user) throw new UserNotFoundError(id);
    return user;
  };

  create = async (data: CreateUserDTO): Promise<UserResponse> => {
    const existing = await this.userRepository.findByEmail(data.email);
    if (existing) throw new EmailAlreadyExistsError(data.email);

    const hashedPassword = await hashPassword(data.password);

    return this.userRepository.create({
      ...data,
      password: hashedPassword,
    });
  };

  update = async (id: string, data: Partial<CreateUserDTO>): Promise<UserResponse> => {
    const user = await this.getById(id);

    if (data.password) {
      data.password = await hashPassword(data.password);
    }

    return this.userRepository.update(user.id, data);
  };

  delete = async (id: string): Promise<void> => {
    await this.getById(id);
    await this.userRepository.delete(id);
  };
}
```

---

## Exemplo: Service (PHP/Laravel)

```php
// app/Services/UserService.php

<?php

declare(strict_types=1);

namespace App\Services;

use App\Repositories\UserRepository;
use App\DTOs\CreateUserDTO;
use App\DTOs\UserResponse;
use App\Exceptions\UserNotFoundException;
use App\Exceptions\EmailAlreadyExistsException;
use Illuminate\Support\Facades\Hash;

class UserService
{
    public function __construct(
        private readonly UserRepository $userRepository,
    ) {}

    public function getById(string $id): UserResponse
    {
        $user = $this->userRepository->findById($id);

        if (!$user) {
            throw new UserNotFoundException($id);
        }

        return UserResponse::from($user);
    }

    public function create(CreateUserDTO $data): UserResponse
    {
        $existing = $this->userRepository->findByEmail($data->email);

        if ($existing) {
            throw new EmailAlreadyExistsException($data->email);
        }

        $user = $this->userRepository->create([
            'name' => $data->name,
            'email' => $data->email,
            'password' => Hash::make($data->password),
        ]);

        return UserResponse::from($user);
    }

    public function update(string $id, array $data): UserResponse
    {
        $this->getById($id);

        if (isset($data['password'])) {
            $data['password'] = Hash::make($data['password']);
        }

        $user = $this->userRepository->update($id, $data);

        return UserResponse::from($user);
    }

    public function delete(string $id): void
    {
        $this->getById($id);
        $this->userRepository->delete($id);
    }
}
```

---

## Exemplo: Erros tipados (Node.js/TypeScript)

```ts
// errors/index.ts

export class AppError extends Error {
  constructor(
    message: string,
    public readonly code: string,
    public readonly statusCode: number,
  ) {
    super(message);
    this.name = this.constructor.name;
  }
}

export class UserNotFoundError extends AppError {
  constructor(id: string) {
    super(`User with id ${id} not found`, 'USER_NOT_FOUND', 404);
  }
}

export class EmailAlreadyExistsError extends AppError {
  constructor(email: string) {
    super(`Email ${email} is already in use`, 'EMAIL_ALREADY_EXISTS', 409);
  }
}

export class UnauthorizedError extends AppError {
  constructor(message = 'Unauthorized') {
    super(message, 'UNAUTHORIZED', 401);
  }
}
```

---

## Exemplo: Erros tipados (PHP/Laravel)

```php
// app/Exceptions/AppException.php

<?php

declare(strict_types=1);

namespace App\Exceptions;

use RuntimeException;

abstract class AppException extends RuntimeException
{
    public function __construct(
        string $message,
        public readonly string $errorCode,
        public readonly int $statusCode,
    ) {
        parent::__construct($message);
    }

    public function toArray(): array
    {
        return [
            'error' => [
                'message' => $this->getMessage(),
                'code' => $this->errorCode,
            ],
        ];
    }
}
```

```php
// app/Exceptions/UserNotFoundException.php

<?php

declare(strict_types=1);

namespace App\Exceptions;

class UserNotFoundException extends AppException
{
    public function __construct(string $id)
    {
        parent::__construct(
            message: "User with id {$id} not found",
            errorCode: 'USER_NOT_FOUND',
            statusCode: 404,
        );
    }
}
```

---

## Exemplo: Repository (Node.js/TypeScript)

```ts
// repositories/user.repository.ts

import { db } from '@/database';
import { users } from '@/models/schema';
import { eq } from 'drizzle-orm';
import type { User, CreateUserInput } from '@/types/user';

export class UserRepository {
  findById = async (id: string): Promise<User | null> => {
    const [user] = await db.select().from(users).where(eq(users.id, id));
    return user ?? null;
  };

  findByEmail = async (email: string): Promise<User | null> => {
    const [user] = await db.select().from(users).where(eq(users.email, email));
    return user ?? null;
  };

  create = async (data: CreateUserInput): Promise<User> => {
    const [user] = await db.insert(users).values(data).returning();
    return user;
  };

  update = async (id: string, data: Partial<CreateUserInput>): Promise<User> => {
    const [user] = await db.update(users).set(data).where(eq(users.id, id)).returning();
    return user;
  };

  delete = async (id: string): Promise<void> => {
    await db.delete(users).where(eq(users.id, id));
  };
}
```

---

## Anti-patterns (o que evitar)

```ts
// ❌ Service conhece HTTP
export class UserService {
  getById = async (req: Request, res: Response) => {
    const user = await this.repo.findById(req.params.id);
    res.json(user);
  };
}

// ✅ Service é agnóstico — controller lida com HTTP
export class UserService {
  getById = async (id: string): Promise<UserResponse> => {
    const user = await this.repo.findById(id);
    if (!user) throw new UserNotFoundError(id);
    return user;
  };
}
```

```ts
// ❌ Lógica de negócio no controller
app.post('/users', async (req, res) => {
  const existing = await db.select().from(users).where(eq(users.email, req.body.email));
  if (existing.length > 0) return res.status(409).json({ error: 'Email exists' });

  const hashed = await bcrypt.hash(req.body.password, 12);
  const user = await db.insert(users).values({ ...req.body, password: hashed }).returning();
  res.status(201).json(user);
});

// ✅ Controller delega para service
app.post('/users', async (req, res) => {
  const data = createUserSchema.parse(req.body);
  const user = await userService.create(data);
  res.status(201).json({ data: user });
});
```

```php
// ❌ Controller gordo — faz tudo
public function store(Request $request): JsonResponse
{
    $validated = $request->validate([...]);
    $existing = User::where('email', $validated['email'])->first();
    if ($existing) return response()->json(['error' => 'exists'], 409);
    $user = User::create([..., 'password' => Hash::make($validated['password'])]);
    return response()->json(['data' => $user], 201);
}

// ✅ Controller magro — delega
public function store(CreateUserRequest $request): JsonResponse
{
    $user = $this->userService->create(CreateUserDTO::from($request->validated()));
    return response()->json(['data' => $user], 201);
}
```
