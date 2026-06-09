# Utils — Frontend

> Guia de boas práticas para criação de funções utilitárias.
> Complementa o `spec.md` deste mesmo diretório.

---

## Regras Gerais [PADRÃO]

- **Declaração:** arrow functions — `export const formatDate = () => {}`
- **Pureza:** sem side effects, sem dependência de estado externo
- **Nomenclatura:** `camelCase` descritivo (`formatDate`, `parseQueryString`, `calculateDiscount`)
- **Localização:** utils globais em `src/utils/` — utils específicas na pasta do módulo

> Para a regra de arquivo único vs pasta e a estrutura de módulos internos, veja
> "Estrutura de Módulos Internos" no [`spec.md`](./spec.md).

---

## Exemplo: Util simples (arquivo único — somente sem testes)

> Este padrão **só é permitido** quando o util não possui testes, tipos separados ou arquivos auxiliares.
> Se o util tiver testes, veja o padrão de pasta em "Estrutura de Módulos Internos" no [`spec.md`](./spec.md).

```ts
// utils/formatDate.ts

export const formatDate = (date: Date | string, locale = 'pt-BR'): string => {
  const parsed = typeof date === 'string' ? new Date(date) : date;
  return parsed.toLocaleDateString(locale, {
    day: '2-digit',
    month: '2-digit',
    year: 'numeric',
  });
};
```

---

## Exemplo: Funções relacionadas no mesmo arquivo

Funções pequenas e fortemente acopladas podem coexistir:

```ts
// utils/currency.ts

export const formatCurrency = (value: number, currency = 'BRL', locale = 'pt-BR'): string => {
  return new Intl.NumberFormat(locale, {
    style: 'currency',
    currency,
  }).format(value);
};

export const parseCurrency = (formatted: string): number => {
  const cleaned = formatted.replace(/[^\d,-]/g, '').replace(',', '.');
  return parseFloat(cleaned);
};

export const centsToCurrency = (cents: number): number => {
  return cents / 100;
};
```

---

## Exemplo: Util com pasta (tipos e testes separados)

```ts
// utils/validation/types.ts

export interface ValidationResult {
  valid: boolean;
  errors: string[];
}

export interface ValidationRule {
  test: (value: string) => boolean;
  message: string;
}
```

```ts
// utils/validation/index.ts

import type { ValidationResult, ValidationRule } from './types';

export const validate = (value: string, rules: ValidationRule[]): ValidationResult => {
  const errors = rules
    .filter((rule) => !rule.test(value))
    .map((rule) => rule.message);

  return { valid: errors.length === 0, errors };
};

export const required: ValidationRule = {
  test: (value) => value.trim().length > 0,
  message: 'Campo obrigatório',
};

export const minLength = (min: number): ValidationRule => ({
  test: (value) => value.length >= min,
  message: `Mínimo de ${min} caracteres`,
});

export const isEmail: ValidationRule = {
  test: (value) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value),
  message: 'Email inválido',
};
```

```ts
// utils/validation/index.test.ts

import { validate, required, minLength, isEmail } from '.';

describe('validate', () => {
  it('returns valid for passing rules', () => {
    const result = validate('hello@test.com', [required, isEmail]);

    expect(result.valid).toBe(true);
    expect(result.errors).toHaveLength(0);
  });

  it('returns errors for failing rules', () => {
    const result = validate('', [required, minLength(3)]);

    expect(result.valid).toBe(false);
    expect(result.errors).toContain('Campo obrigatório');
    expect(result.errors).toContain('Mínimo de 3 caracteres');
  });
});
```

---

## Exemplo: Util para manipulação de arrays

```ts
// utils/array.ts

export const groupBy = <T>(items: T[], key: keyof T): Record<string, T[]> => {
  return items.reduce((groups, item) => {
    const groupKey = String(item[key]);
    return {
      ...groups,
      [groupKey]: [...(groups[groupKey] || []), item],
    };
  }, {} as Record<string, T[]>);
};

export const uniqueBy = <T>(items: T[], key: keyof T): T[] => {
  const seen = new Set<T[keyof T]>();
  return items.filter((item) => {
    if (seen.has(item[key])) return false;
    seen.add(item[key]);
    return true;
  });
};
```

---

## Exemplo: Util para tratamento de erros

```ts
// utils/error.ts

export interface AppError {
  message: string;
  code: string;
  statusCode?: number;
}

export const isAppError = (error: unknown): error is AppError => {
  return (
    typeof error === 'object' &&
    error !== null &&
    'message' in error &&
    'code' in error
  );
};

export const getErrorMessage = (error: unknown): string => {
  if (isAppError(error)) return error.message;
  if (error instanceof Error) return error.message;
  return 'Ocorreu um erro inesperado';
};
```

---

## Anti-patterns (o que evitar)

```ts
// ❌ Util com side effects
export const saveToStorage = (key: string, value: unknown) => {
  localStorage.setItem(key, JSON.stringify(value));
  console.log('Saved:', key);
};

// ✅ Util pura — side effects ficam em hooks ou services
export const serialize = (value: unknown): string => {
  return JSON.stringify(value);
};
```

```ts
// ❌ Sem tipagem de retorno
export const calculateTotal = (items) => {
  return items.reduce((sum, item) => sum + item.price, 0);
};

// ✅ Tipagem explícita
interface CartItem {
  id: string;
  price: number;
  quantity: number;
}

export const calculateTotal = (items: CartItem[]): number => {
  return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
};
```

```ts
// ❌ Util que depende de estado global
let cachedResult: string | null = null;

export const getConfig = () => {
  if (cachedResult) return cachedResult;
  // ...
};

// ✅ Util pura — cache fica em quem chama
export const buildConfig = (env: Record<string, string>): AppConfig => {
  return {
    apiUrl: env.API_URL ?? 'http://localhost:3000',
    debug: env.NODE_ENV === 'development',
  };
};
```
