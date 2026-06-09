# Migrations — Banco de Dados

> Guia de boas práticas para migrations, seeds e alterações seguras de schema.
> Complementa o [`spec.md`](./spec.md) deste mesmo diretório.

---

## Regras Gerais [PADRÃO]

- **Obrigatoriedade:** toda alteração de schema passa por migration — proibido alterar banco diretamente
- **Irreversibilidade:** migrations são irreversíveis por padrão — `down()` opcional e documentado
- **Nomenclatura:** nome descreve a mudança (`add_deleted_at_to_orders`, `create_user_roles_table`)
- **Conteúdo:** migrations contêm apenas DDL e DML simples — proibido lógica de negócio
- **Teste:** toda migration testada em desenvolvimento antes de staging
- **Idempotência:** data migrations são idempotentes — rodar duas vezes produz o mesmo resultado
- **Separação:** alterações de schema e data migrations são commits separados

> Para convenções de nomenclatura de tabelas, colunas e índices, veja "Nomenclatura" no [`spec.md`](./spec.md).
> Para exemplos de schema design, veja [`modeling.md`](./modeling.md).

---

## Exemplo: Migration de criação (SQL)

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
```

---

## Exemplo: Migration de criação (Drizzle Kit)

```ts
// db/schema/users.ts — schema define a tabela (veja modeling.md)

// Gerar migration automaticamente:
// npx drizzle-kit generate

// Aplicar migration:
// npx drizzle-kit migrate
```

```sql
-- drizzle/0001_create_users.sql (gerado automaticamente)

CREATE TABLE "users" (
  "id" uuid PRIMARY KEY DEFAULT gen_random_uuid() NOT NULL,
  "name" varchar(100) NOT NULL,
  "email" varchar(255) NOT NULL,
  "password_hash" varchar(255) NOT NULL,
  "role" varchar(20) DEFAULT 'user' NOT NULL,
  "deleted_at" timestamp with time zone,
  "created_at" timestamp with time zone DEFAULT now() NOT NULL,
  "updated_at" timestamp with time zone DEFAULT now() NOT NULL,
  CONSTRAINT "users_email_unique" UNIQUE("email")
);

CREATE INDEX "idx_users_email" ON "users" USING btree ("email");
```

> Drizzle Kit gera SQL a partir do schema TypeScript. Revisar o SQL gerado antes de aplicar — nunca confiar cegamente no gerador.

---

## Exemplo: Migration de criação (Laravel)

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
        });
    }
};
```

```bash
# Gerar migration:
php artisan make:migration create_users_table

# Aplicar:
php artisan migrate

# Rollback (quando down() existe):
php artisan migrate:rollback
```

---

## Exemplo: Alteração segura de coluna (ADD COLUMN)

Adicionar coluna a tabela existente em produção requer cuidado. A sequência segura:

```
1. ADD COLUMN nullable (sem default pesado)  → migration 1
2. Backfill de dados (preencher valores)     → migration 2 (data migration)
3. ALTER COLUMN SET NOT NULL (se necessário)  → migration 3
```

### Passo 1 — Adicionar coluna nullable

```sql
-- migrations/010_add_phone_to_users.sql

ALTER TABLE users ADD COLUMN phone VARCHAR(20);
```

```ts
// Drizzle — adicionar campo ao schema e gerar migration
// db/schema/users.ts
phone: varchar('phone', { length: 20 }),
```

```php
// database/migrations/2025_03_15_000001_add_phone_to_users.php

return new class extends Migration
{
    public function up(): void
    {
        Schema::table('users', function (Blueprint $table) {
            $table->string('phone', 20)->nullable();
        });
    }
};
```

### Passo 2 — Backfill (data migration separada)

```sql
-- migrations/011_backfill_phone_on_users.sql

UPDATE users SET phone = 'unknown' WHERE phone IS NULL;
```

### Passo 3 — Tornar NOT NULL (após backfill confirmado)

```sql
-- migrations/012_set_phone_not_null_on_users.sql

ALTER TABLE users ALTER COLUMN phone SET NOT NULL;
ALTER TABLE users ALTER COLUMN phone SET DEFAULT 'unknown';
```

> Nunca faça os três passos numa única migration. Se o backfill falhar, a coluna NOT NULL impede inserções e causa downtime.

---

## Exemplo: Renomear coluna (alteração destrutiva)

Renomear colunas em produção é uma operação de risco. A sequência segura com zero downtime:

```
1. ADD nova coluna                           → migration
2. Dual-write (app escreve em ambas)         → deploy de código
3. Backfill coluna nova com dados da antiga  → data migration
4. Mudar reads para coluna nova              → deploy de código
5. Remover dual-write                        → deploy de código
6. DROP coluna antiga                        → migration
```

> Para tabelas pequenas ou em staging, `ALTER TABLE RENAME COLUMN` é aceitável diretamente.

---

## Exemplo: Seeds para desenvolvimento

Seeds populam o banco com dados consistentes para desenvolvimento e testes.

```sql
-- seeds/001_users.sql

INSERT INTO users (id, name, email, password_hash, role) VALUES
  ('550e8400-e29b-41d4-a716-446655440001', 'Admin User', 'admin@dev.local', '$2b$10$hash1', 'admin'),
  ('550e8400-e29b-41d4-a716-446655440002', 'Test User', 'user@dev.local', '$2b$10$hash2', 'user')
ON CONFLICT (email) DO NOTHING;
```

```ts
// db/seeds/users.ts (Drizzle)

import { db } from '@/db';
import { users } from '@/db/schema';

export const seedUsers = async () => {
  await db.insert(users).values([
    { name: 'Admin User', email: 'admin@dev.local', passwordHash: '$2b$10$hash1', role: 'admin' },
    { name: 'Test User', email: 'user@dev.local', passwordHash: '$2b$10$hash2', role: 'user' },
  ]).onConflictDoNothing({ target: users.email });
};
```

```php
// database/seeders/UserSeeder.php (Laravel)

<?php

declare(strict_types=1);

namespace Database\Seeders;

use App\Models\User;
use Illuminate\Database\Seeder;

class UserSeeder extends Seeder
{
    public function run(): void
    {
        User::factory()->create([
            'name' => 'Admin User',
            'email' => 'admin@dev.local',
            'role' => 'admin',
        ]);

        User::factory()->count(10)->create();
    }
}
```

---

## Exemplo: Factories para testes (Laravel)

```php
// database/factories/UserFactory.php

<?php

declare(strict_types=1);

namespace Database\Factories;

use Illuminate\Database\Eloquent\Factories\Factory;
use Illuminate\Support\Facades\Hash;

class UserFactory extends Factory
{
    public function definition(): array
    {
        return [
            'name' => fake()->name(),
            'email' => fake()->unique()->safeEmail(),
            'password_hash' => Hash::make('password'),
            'role' => 'user',
        ];
    }

    public function admin(): static
    {
        return $this->state(['role' => 'admin']);
    }
}
```

```php
// Uso nos testes
$user = User::factory()->create();
$admin = User::factory()->admin()->create();
$users = User::factory()->count(5)->create();
```

---

## Exemplo: Rollback — quando e como

Rollback (`down()`) é opcional e deve ser documentado quando presente. Nem toda migration é revertível com segurança.

### Revertível (CREATE TABLE, ADD COLUMN)

```php
return new class extends Migration
{
    public function up(): void
    {
        Schema::create('categories', function (Blueprint $table) {
            $table->uuid('id')->primary();
            $table->string('name', 100)->unique();
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('categories');
    }
};
```

### Não revertível (data migration, DROP COLUMN com dados)

```php
return new class extends Migration
{
    public function up(): void
    {
        Schema::table('users', function (Blueprint $table) {
            $table->dropColumn('legacy_role');
        });
    }

    // down() intencionalmente ausente — dados perdidos não são recuperáveis
};
```

> **Regra:** se `down()` pode causar perda de dados, não implemente. Documente no PR por que a migration é irreversível.

---

## Anti-patterns (o que evitar)

```sql
-- ❌ Alterar banco diretamente em produção
ALTER TABLE users ADD COLUMN phone VARCHAR(20) NOT NULL;
-- Executado direto no psql, sem registro, sem rastreabilidade

-- ✅ Toda mudança via migration versionada no repositório
-- migrations/010_add_phone_to_users.sql
ALTER TABLE users ADD COLUMN phone VARCHAR(20);
```

```php
// ❌ Lógica de negócio dentro da migration
return new class extends Migration
{
    public function up(): void
    {
        Schema::create('orders', function (Blueprint $table) {
            $table->uuid('id')->primary();
            // ...
        });

        // Lógica de negócio NÃO pertence aqui
        $users = DB::table('users')->where('role', 'admin')->get();
        foreach ($users as $user) {
            Notification::send($user, new MigrationComplete());
        }
    }
};

// ✅ Migration contém apenas DDL/DML — lógica de negócio fica na aplicação
return new class extends Migration
{
    public function up(): void
    {
        Schema::create('orders', function (Blueprint $table) {
            $table->uuid('id')->primary();
            // ...
        });
    }
};
```

```sql
-- ❌ ADD COLUMN NOT NULL sem default em tabela com dados
ALTER TABLE users ADD COLUMN phone VARCHAR(20) NOT NULL;
-- Falha: coluna NOT NULL sem default em tabela populada

-- ✅ Sequência segura: nullable → backfill → NOT NULL
ALTER TABLE users ADD COLUMN phone VARCHAR(20);
UPDATE users SET phone = 'unknown' WHERE phone IS NULL;
ALTER TABLE users ALTER COLUMN phone SET NOT NULL;
```

```sql
-- ❌ Data migration não-idempotente
UPDATE users SET credits = credits + 100;
-- Se rodar duas vezes, dá 200 de crédito em vez de 100

-- ✅ Data migration idempotente
UPDATE users SET credits = 100 WHERE credits = 0;
-- Rodar duas vezes produz o mesmo resultado
```

```sql
-- ❌ Migration destrutiva sem backup
DROP TABLE user_sessions;
-- Dados perdidos permanentemente

-- ✅ Renomear antes de dropar — período de segurança
ALTER TABLE user_sessions RENAME TO user_sessions_deprecated;
-- Após confirmar que nada usa a tabela (7-30 dias):
-- DROP TABLE user_sessions_deprecated;
```
