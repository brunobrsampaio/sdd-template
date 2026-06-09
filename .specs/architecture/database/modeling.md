# Modelagem — Banco de Dados

> Guia de boas práticas para design de schema, tipos, relacionamentos e constraints.
> Complementa o [`spec.md`](./spec.md) deste mesmo diretório.

---

## Regras Gerais [PADRÃO]

- **Timestamps:** toda tabela tem `created_at` e `updated_at` — preenchidos automaticamente
- **Chave primária:** sempre `id` do tipo UUID — proibido IDs sequenciais em tabelas expostas via API
- **Soft delete:** preferir `deleted_at` sobre exclusão física — avaliar caso a caso
- **Constraints:** toda regra de integridade expressável no banco deve ser constraint — não confiar apenas na aplicação
- **Nullable:** colunas são `NOT NULL` por padrão — `NULL` apenas quando a ausência de valor tem significado de negócio
- **Defaults:** valores padrão definidos no banco quando possível — reduz erros de inserção incompleta

> Para convenções de nomenclatura (tabelas, colunas, índices, constraints), veja "Nomenclatura" no [`spec.md`](./spec.md).

---

## Exemplo: Schema base com timestamps (SQL)

```sql
-- migrations/001_create_users.sql

CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(100) NOT NULL,
  email VARCHAR(255) NOT NULL UNIQUE,
  password_hash VARCHAR(255) NOT NULL,
  role VARCHAR(20) NOT NULL DEFAULT 'user',
  deleted_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_users_email ON users (email);
CREATE INDEX idx_users_role ON users (role) WHERE deleted_at IS NULL;
```

---

## Exemplo: Schema base com timestamps (Node.js/TypeScript — Drizzle ORM)

```ts
// db/schema/users.ts

import { pgTable, uuid, varchar, timestamp, index } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
  id: uuid('id').primaryKey().defaultRandom(),
  name: varchar('name', { length: 100 }).notNull(),
  email: varchar('email', { length: 255 }).notNull().unique(),
  passwordHash: varchar('password_hash', { length: 255 }).notNull(),
  role: varchar('role', { length: 20 }).notNull().default('user'),
  deletedAt: timestamp('deleted_at', { withTimezone: true }),
  createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).notNull().defaultNow(),
}, (table) => [
  index('idx_users_email').on(table.email),
  index('idx_users_role').on(table.role),
]);
```

---

## Exemplo: Schema base com timestamps (PHP/Laravel — Eloquent)

```php
// database/migrations/2025_01_01_000001_create_users_table.php

<?php

declare(strict_types=1);

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('users', function (Blueprint $table) {
            $table->uuid('id')->primary();
            $table->string('name', 100);
            $table->string('email', 255)->unique();
            $table->string('password_hash', 255);
            $table->string('role', 20)->default('user');
            $table->softDeletes();
            $table->timestamps();

            $table->index('email');
            $table->index('role');
        });
    }
};
```

```php
// app/Models/User.php

<?php

declare(strict_types=1);

namespace App\Models;

use Illuminate\Database\Eloquent\Concerns\HasUuids;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class User extends Model
{
    use HasUuids, SoftDeletes;

    protected $fillable = [
        'name',
        'email',
        'password_hash',
        'role',
    ];

    protected $hidden = [
        'password_hash',
    ];
}
```

---

## Exemplo: Relacionamento 1:N (SQL)

```sql
-- migrations/002_create_orders.sql

CREATE TABLE orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
  status VARCHAR(20) NOT NULL DEFAULT 'pending',
  total_cents INTEGER NOT NULL,
  deleted_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  CONSTRAINT chk_orders_status CHECK (status IN ('pending', 'paid', 'shipped', 'cancelled')),
  CONSTRAINT chk_orders_total_positive CHECK (total_cents >= 0)
);

CREATE INDEX idx_orders_user_id ON orders (user_id);
CREATE INDEX idx_orders_status ON orders (status) WHERE deleted_at IS NULL;
```

---

## Exemplo: Relacionamento 1:N (Drizzle ORM)

```ts
// db/schema/orders.ts

import { pgTable, uuid, varchar, integer, timestamp, index, check } from 'drizzle-orm/pg-core';
import { sql } from 'drizzle-orm';
import { users } from './users';

export const orders = pgTable('orders', {
  id: uuid('id').primaryKey().defaultRandom(),
  userId: uuid('user_id').notNull().references(() => users.id, { onDelete: 'restrict' }),
  status: varchar('status', { length: 20 }).notNull().default('pending'),
  totalCents: integer('total_cents').notNull(),
  deletedAt: timestamp('deleted_at', { withTimezone: true }),
  createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).notNull().defaultNow(),
}, (table) => [
  index('idx_orders_user_id').on(table.userId),
  index('idx_orders_status').on(table.status),
  check('chk_orders_status', sql`${table.status} IN ('pending', 'paid', 'shipped', 'cancelled')`),
  check('chk_orders_total_positive', sql`${table.totalCents} >= 0`),
]);
```

---

## Exemplo: Relacionamento 1:N (Eloquent)

```php
// database/migrations/2025_01_01_000002_create_orders_table.php

<?php

declare(strict_types=1);

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('orders', function (Blueprint $table) {
            $table->uuid('id')->primary();
            $table->foreignUuid('user_id')->constrained()->restrictOnDelete();
            $table->string('status', 20)->default('pending');
            $table->integer('total_cents');
            $table->softDeletes();
            $table->timestamps();

            $table->index('user_id');
            $table->index('status');
        });
    }
};
```

```php
// app/Models/Order.php — relacionamento no modelo

<?php

declare(strict_types=1);

namespace App\Models;

use Illuminate\Database\Eloquent\Concerns\HasUuids;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
use Illuminate\Database\Eloquent\SoftDeletes;

class Order extends Model
{
    use HasUuids, SoftDeletes;

    protected $fillable = ['user_id', 'status', 'total_cents'];

    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
}
```

```php
// app/Models/User.php — lado inverso

public function orders(): HasMany
{
    return $this->hasMany(Order::class);
}
```

---

## Exemplo: Relacionamento N:N com dados na junção (SQL)

Tabelas de junção seguem a convenção `<tabela_a>_<tabela_b>` em ordem alfabética. Quando a junção carrega dados próprios, ela é uma entidade de domínio.

```sql
-- migrations/003_create_order_items.sql

CREATE TABLE order_items (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_id UUID NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
  product_id UUID NOT NULL REFERENCES products(id) ON DELETE RESTRICT,
  quantity INTEGER NOT NULL,
  unit_price_cents INTEGER NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  CONSTRAINT chk_order_items_quantity CHECK (quantity > 0),
  CONSTRAINT chk_order_items_price CHECK (unit_price_cents >= 0),
  UNIQUE (order_id, product_id)
);

CREATE INDEX idx_order_items_order_id ON order_items (order_id);
CREATE INDEX idx_order_items_product_id ON order_items (product_id);
```

---

## Exemplo: Soft delete — query scoping

Soft delete exige que todas as queries filtrem registros deletados por padrão. ORMs geralmente fazem isso automaticamente.

```sql
-- SQL puro — filtro manual obrigatório
SELECT id, name, email FROM users WHERE deleted_at IS NULL;

-- Índice parcial para queries frequentes (exclui deletados do índice)
CREATE INDEX idx_users_email_active ON users (email) WHERE deleted_at IS NULL;
```

```ts
// Drizzle ORM — helper para filtrar soft-deleted
import { isNull } from 'drizzle-orm';

const activeUsers = await db
  .select()
  .from(users)
  .where(isNull(users.deletedAt));

// Soft delete — nunca DELETE
await db
  .update(users)
  .set({ deletedAt: new Date() })
  .where(eq(users.id, userId));
```

```php
// Eloquent — SoftDeletes filtra automaticamente
$activeUsers = User::all(); // exclui deletados por padrão
$allUsers = User::withTrashed()->get(); // inclui deletados
$user->delete(); // SET deleted_at = now()
$user->forceDelete(); // DELETE real — usar com cautela
```

---

## Exemplo: Enum vs tabela de lookup

### Quando usar enum no banco

Valores verdadeiramente fixos que nunca mudam (ou mudam com deploy):

```sql
-- Status de pedido — conjunto fechado e estável
ALTER TABLE orders ADD CONSTRAINT chk_orders_status
  CHECK (status IN ('pending', 'paid', 'shipped', 'cancelled'));
```

### Quando usar tabela de lookup

Valores que podem mudar em runtime (categorias, tags, permissões):

```sql
CREATE TABLE categories (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(100) NOT NULL UNIQUE,
  slug VARCHAR(100) NOT NULL UNIQUE,
  active BOOLEAN NOT NULL DEFAULT true,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Referência por FK em vez de string solta
ALTER TABLE products ADD COLUMN category_id UUID REFERENCES categories(id);
```

> **Regra prática:** se um PM pode pedir para adicionar ou remover um valor sem deploy, é tabela de lookup. Se a mudança exige alteração de código, enum/check é aceitável.

---

## Anti-patterns (o que evitar)

```sql
-- ❌ Tabela sem chave primária
CREATE TABLE logs (
  message TEXT,
  created_at TIMESTAMPTZ
);

-- ✅ Toda tabela tem PK — mesmo tabelas de log
CREATE TABLE logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  message TEXT NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

```sql
-- ❌ Colunas genéricas sem significado
CREATE TABLE users (
  id UUID PRIMARY KEY,
  data1 VARCHAR(255),
  data2 VARCHAR(255),
  type INTEGER,
  flag BOOLEAN
);

-- ✅ Nomes que revelam intenção
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  display_name VARCHAR(100) NOT NULL,
  bio VARCHAR(500),
  role VARCHAR(20) NOT NULL DEFAULT 'user',
  email_verified BOOLEAN NOT NULL DEFAULT false
);
```

```sql
-- ❌ JSON para tudo — perde integridade referencial e tipagem
CREATE TABLE orders (
  id UUID PRIMARY KEY,
  data JSONB -- contém user_id, items, status, tudo misturado
);

-- ✅ JSON apenas para dados verdadeiramente flexíveis (metadata, preferências)
CREATE TABLE orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id),
  status VARCHAR(20) NOT NULL,
  total_cents INTEGER NOT NULL,
  metadata JSONB DEFAULT '{}'
);
```

```sql
-- ❌ Coluna nullable sem razão — NULL vira "valor padrão silencioso"
CREATE TABLE products (
  id UUID PRIMARY KEY,
  name VARCHAR(100),     -- pode ser NULL? nome é obrigatório
  price INTEGER          -- pode ser NULL? preço é obrigatório
);

-- ✅ NOT NULL por padrão — nullable apenas com significado de negócio
CREATE TABLE products (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(100) NOT NULL,
  price_cents INTEGER NOT NULL,
  discount_ends_at TIMESTAMPTZ  -- NULL = sem desconto ativo (significado claro)
);
```

```sql
-- ❌ Sem constraint — regra de negócio existe apenas na aplicação
CREATE TABLE orders (
  id UUID PRIMARY KEY,
  total_cents INTEGER,
  status VARCHAR(50)
);

-- ✅ Constraints protegem a integridade mesmo fora da aplicação
CREATE TABLE orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  total_cents INTEGER NOT NULL,
  status VARCHAR(20) NOT NULL DEFAULT 'pending',
  CONSTRAINT chk_orders_total_positive CHECK (total_cents >= 0),
  CONSTRAINT chk_orders_status CHECK (status IN ('pending', 'paid', 'shipped', 'cancelled'))
);
```
