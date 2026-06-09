# Queries e Acesso a Dados — Banco de Dados

> Guia de boas práticas para repositórios, queries, transações e performance.
> Complementa o [`spec.md`](./spec.md) deste mesmo diretório.

---

## Regras Gerais [PADRÃO]

- **Repositório:** único ponto de acesso ao banco — proibido query direta em services ou controllers
- **SELECT:** proibido `SELECT *` — sempre listar colunas explicitamente
- **Transações:** obrigatórias quando múltiplas operações precisam ser atômicas
- **Paginação:** obrigatória em queries que podem retornar conjuntos grandes
- **N+1:** proibido — usar eager loading ou JOINs
- **Queries em loop:** proibido — usar batch ou `IN`
- **Comentários:** queries com JOIN complexo ou subquery são documentadas com comentário explicativo

> Para regras de índices e análise de performance, veja "Performance" no [`spec.md`](./spec.md).
> Para design de schema e relacionamentos, veja [`modeling.md`](./modeling.md).

---

## Exemplo: Repository CRUD (Node.js/TypeScript — Drizzle ORM)

```ts
// repositories/user.repository.ts

import { eq, isNull } from 'drizzle-orm';
import { db } from '@/db';
import { users } from '@/db/schema';

interface CreateUserData {
  name: string;
  email: string;
  passwordHash: string;
  role?: string;
}

interface UpdateUserData {
  name?: string;
  email?: string;
  role?: string;
}

export const userRepository = {
  async findById(id: string) {
    const [user] = await db
      .select({
        id: users.id,
        name: users.name,
        email: users.email,
        role: users.role,
        createdAt: users.createdAt,
      })
      .from(users)
      .where(eq(users.id, id))
      .limit(1);

    return user ?? null;
  },

  async findByEmail(email: string) {
    const [user] = await db
      .select({
        id: users.id,
        name: users.name,
        email: users.email,
        passwordHash: users.passwordHash,
        role: users.role,
      })
      .from(users)
      .where(eq(users.email, email))
      .limit(1);

    return user ?? null;
  },

  async create(data: CreateUserData) {
    const [user] = await db
      .insert(users)
      .values(data)
      .returning({
        id: users.id,
        name: users.name,
        email: users.email,
        role: users.role,
        createdAt: users.createdAt,
      });

    return user;
  },

  async update(id: string, data: UpdateUserData) {
    const [user] = await db
      .update(users)
      .set({ ...data, updatedAt: new Date() })
      .where(eq(users.id, id))
      .returning({
        id: users.id,
        name: users.name,
        email: users.email,
        role: users.role,
        updatedAt: users.updatedAt,
      });

    return user ?? null;
  },

  async softDelete(id: string) {
    await db
      .update(users)
      .set({ deletedAt: new Date() })
      .where(eq(users.id, id));
  },
};
```

---

## Exemplo: Repository CRUD (PHP/Laravel — Eloquent)

```php
// app/Repositories/UserRepository.php

<?php

declare(strict_types=1);

namespace App\Repositories;

use App\Models\User;
use Illuminate\Database\Eloquent\Collection;

class UserRepository
{
    public function findById(string $id): ?User
    {
        return User::select(['id', 'name', 'email', 'role', 'created_at'])
            ->find($id);
    }

    public function findByEmail(string $email): ?User
    {
        return User::select(['id', 'name', 'email', 'password_hash', 'role'])
            ->where('email', $email)
            ->first();
    }

    public function create(array $data): User
    {
        return User::create($data);
    }

    public function update(string $id, array $data): ?User
    {
        $user = User::find($id);

        if (!$user) {
            return null;
        }

        $user->update($data);
        return $user->fresh(['id', 'name', 'email', 'role', 'updated_at']);
    }

    public function softDelete(string $id): void
    {
        User::where('id', $id)->delete();
    }
}
```

---

## Exemplo: Paginação — cursor-based vs offset+limit

### Offset+limit (simples, com limitações)

Adequado para conjuntos pequenos ou quando o usuário precisa navegar para páginas arbitrárias.

```ts
// Drizzle ORM — offset+limit
const page = 1;
const perPage = 20;

const [items, [{ total }]] = await Promise.all([
  db.select({
    id: orders.id,
    status: orders.status,
    totalCents: orders.totalCents,
    createdAt: orders.createdAt,
  })
    .from(orders)
    .where(eq(orders.userId, userId))
    .orderBy(desc(orders.createdAt))
    .limit(perPage)
    .offset((page - 1) * perPage),

  db.select({ total: count() })
    .from(orders)
    .where(eq(orders.userId, userId)),
]);

// Resposta padronizada
const result = {
  data: items,
  meta: { page, per_page: perPage, total: total, last_page: Math.ceil(total / perPage) },
};
```

```php
// Eloquent — paginação built-in
$orders = Order::select(['id', 'status', 'total_cents', 'created_at'])
    ->where('user_id', $userId)
    ->orderByDesc('created_at')
    ->paginate(perPage: 20);
```

### Cursor-based (performante, para conjuntos grandes)

Usa o último item como referência para a próxima página. Mais eficiente que offset em tabelas grandes.

```ts
// Drizzle ORM — cursor-based
const cursor = '2025-01-15T14:30:00Z'; // createdAt do último item da página anterior
const perPage = 20;

const items = await db
  .select({
    id: orders.id,
    status: orders.status,
    totalCents: orders.totalCents,
    createdAt: orders.createdAt,
  })
  .from(orders)
  .where(
    and(
      eq(orders.userId, userId),
      lt(orders.createdAt, new Date(cursor)),
    )
  )
  .orderBy(desc(orders.createdAt))
  .limit(perPage + 1); // +1 para saber se existe próxima página

const hasMore = items.length > perPage;
const data = hasMore ? items.slice(0, -1) : items;
const nextCursor = hasMore ? data[data.length - 1].createdAt.toISOString() : null;
```

```php
// Eloquent — cursor pagination built-in
$orders = Order::select(['id', 'status', 'total_cents', 'created_at'])
    ->where('user_id', $userId)
    ->orderByDesc('created_at')
    ->cursorPaginate(perPage: 20);
```

> **Regra:** use offset+limit para admin panels e listagens com navegação por página. Use cursor-based para feeds, timelines e APIs com scroll infinito.

---

## Exemplo: Transações

```ts
// Drizzle ORM — transação com rollback automático em caso de erro
const newOrder = await db.transaction(async (tx) => {
  const [order] = await tx
    .insert(orders)
    .values({ userId, status: 'pending', totalCents })
    .returning();

  await tx
    .insert(orderItems)
    .values(items.map((item) => ({
      orderId: order.id,
      productId: item.productId,
      quantity: item.quantity,
      unitPriceCents: item.unitPriceCents,
    })));

  await tx
    .update(products)
    .set({ stock: sql`stock - ${item.quantity}` })
    .where(eq(products.id, item.productId));

  return order;
});
```

```php
// Eloquent — transação com closure (rollback automático em exceção)
$order = DB::transaction(function () use ($userId, $totalCents, $items) {
    $order = Order::create([
        'user_id' => $userId,
        'status' => 'pending',
        'total_cents' => $totalCents,
    ]);

    foreach ($items as $item) {
        OrderItem::create([
            'order_id' => $order->id,
            'product_id' => $item['product_id'],
            'quantity' => $item['quantity'],
            'unit_price_cents' => $item['unit_price_cents'],
        ]);

        Product::where('id', $item['product_id'])
            ->decrement('stock', $item['quantity']);
    }

    return $order;
});
```

> Transações garantem atomicidade — se qualquer operação falhar, todas são revertidas. Use sempre que criar/modificar registros em múltiplas tabelas.

---

## Exemplo: Evitando N+1

O problema N+1 ocorre quando uma query dispara N queries adicionais dentro de um loop.

```ts
// ❌ N+1 — 1 query para orders + N queries para users
const orders = await db.select().from(ordersTable);
for (const order of orders) {
  const [user] = await db.select().from(users).where(eq(users.id, order.userId));
  order.user = user;
}

// ✅ JOIN — 1 query para tudo
const ordersWithUser = await db
  .select({
    orderId: ordersTable.id,
    status: ordersTable.status,
    totalCents: ordersTable.totalCents,
    userName: users.name,
    userEmail: users.email,
  })
  .from(ordersTable)
  .innerJoin(users, eq(ordersTable.userId, users.id));
```

```php
// ❌ N+1 — Eloquent carrega users um a um
$orders = Order::all();
foreach ($orders as $order) {
    echo $order->user->name; // query por iteração
}

// ✅ Eager loading — 2 queries no total (orders + users)
$orders = Order::with('user')->get();
foreach ($orders as $order) {
    echo $order->user->name; // já carregado
}

// ✅ Alternativa: JOIN explícito quando precisa de menos dados
$orders = Order::select('orders.*', 'users.name as user_name')
    ->join('users', 'orders.user_id', '=', 'users.id')
    ->get();
```

---

## Exemplo: Batch operations

```ts
// ❌ Insert em loop — N queries
for (const item of items) {
  await db.insert(orderItems).values(item);
}

// ✅ Batch insert — 1 query
await db.insert(orderItems).values(items);
```

```php
// ❌ Insert em loop — N queries
foreach ($items as $item) {
    OrderItem::create($item);
}

// ✅ Batch insert — 1 query
OrderItem::insert($items);

// ✅ Upsert em lote
OrderItem::upsert(
    $items,
    ['order_id', 'product_id'],
    ['quantity', 'unit_price_cents']
);
```

```sql
-- SQL puro — batch insert
INSERT INTO order_items (order_id, product_id, quantity, unit_price_cents)
VALUES
  ('ord-1', 'prod-1', 2, 1500),
  ('ord-1', 'prod-2', 1, 3000),
  ('ord-1', 'prod-3', 3, 500);
```

---

## Exemplo: EXPLAIN ANALYZE — analisando queries lentas

Toda query acima de 100ms em produção deve ser investigada com `EXPLAIN ANALYZE` antes de otimizar.

```sql
-- Verificar plano de execução
EXPLAIN ANALYZE
SELECT o.id, o.status, o.total_cents, u.name
FROM orders o
INNER JOIN users u ON o.user_id = u.id
WHERE o.status = 'pending'
  AND o.created_at > '2025-01-01'
ORDER BY o.created_at DESC
LIMIT 20;
```

### O que procurar no resultado

| Sinal | Significado | Ação |
|-------|------------|------|
| `Seq Scan` em tabela grande | Full table scan — sem índice | Criar índice na coluna filtrada |
| `Sort` com alto custo | Ordenação em memória/disco | Criar índice que cubra a ordenação |
| `Nested Loop` com muitas rows | JOIN ineficiente | Verificar índices nas FKs |
| `Hash Join` com `Rows Removed by Filter` alto | Muitas linhas descartadas | Filtro mais restritivo ou índice parcial |
| `actual rows` >> `estimated rows` | Estatísticas desatualizadas | `ANALYZE tabela` |

```sql
-- Criar índice para resolver Seq Scan
CREATE INDEX idx_orders_status_created ON orders (status, created_at DESC)
  WHERE deleted_at IS NULL;

-- Atualizar estatísticas
ANALYZE orders;
```

---

## Anti-patterns (o que evitar)

```sql
-- ❌ SELECT * — carrega colunas desnecessárias
SELECT * FROM users WHERE id = '123';

-- ✅ Listar apenas colunas necessárias
SELECT id, name, email, role FROM users WHERE id = '123';
```

```ts
// ❌ Query direta no service — bypassa repositório
export class UserService {
  async getUser(id: string) {
    const [user] = await db.select().from(users).where(eq(users.id, id));
    return user;
  }
}

// ✅ Service delega para repositório
export class UserService {
  constructor(private readonly userRepo: typeof userRepository) {}

  async getUser(id: string) {
    return this.userRepo.findById(id);
  }
}
```

```ts
// ❌ SQL por concatenação de strings — vulnerável a SQL injection
const query = `SELECT * FROM users WHERE email = '${email}'`;

// ✅ Prepared statements / parâmetros
const [user] = await db
  .select()
  .from(users)
  .where(eq(users.email, email));

// ✅ SQL puro com parâmetros
const result = await pool.query('SELECT id, name FROM users WHERE email = $1', [email]);
```

```ts
// ❌ Transação implícita — operações não-atômicas
const order = await db.insert(orders).values({ userId, totalCents }).returning();
await db.insert(orderItems).values(items); // se falhar, order fica órfã
await db.update(products).set({ stock: sql`stock - 1` }); // se falhar, estoque inconsistente

// ✅ Transação explícita — tudo ou nada
await db.transaction(async (tx) => {
  const [order] = await tx.insert(orders).values({ userId, totalCents }).returning();
  await tx.insert(orderItems).values(items);
  await tx.update(products).set({ stock: sql`stock - 1` });
});
```

```sql
-- ❌ Paginação sem limite — retorna a tabela inteira
SELECT id, name FROM users ORDER BY created_at;

-- ✅ Paginação obrigatória em queries de listagem
SELECT id, name FROM users ORDER BY created_at DESC LIMIT 20 OFFSET 0;
```

```php
// ❌ Filtro em código — traz tudo do banco e filtra no PHP
$allOrders = Order::all();
$pending = $allOrders->filter(fn ($o) => $o->status === 'pending');

// ✅ Filtro no banco — traz apenas o necessário
$pending = Order::where('status', 'pending')->get();
```
