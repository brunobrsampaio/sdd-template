# Componentes — Frontend

> Guia de boas práticas para criação de componentes React.
> Complementa o [`spec.md`](./spec.md) deste mesmo diretório.

---

## Regras Gerais [PADRÃO]

- **Props:** sempre tipadas com interface explícita
- **Prop drilling:** proibido além de 2 níveis — use contexto ou estado global
- **Declaração:** arrow functions — proibido class components e function declarations
- **Side effects:** ficam em hooks, não diretamente nos componentes
- **Arquivo:** um componente por arquivo — o `index.tsx` é o ponto de entrada único
- **JSX:** proibido lógica de negócio dentro do JSX — extraia para variáveis ou hooks

---

## Exemplo: Componente simples (sem estado)

```tsx
// components/Button/index.tsx

interface ButtonProps {
  label: string;
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
  onClick: () => void;
}

export const Button = ({ label, variant = 'primary', disabled = false, onClick }: ButtonProps) => {
  const baseStyles = 'px-4 py-2 rounded font-medium transition-colors';
  const variantStyles = variant === 'primary'
    ? 'bg-blue-600 text-white hover:bg-blue-700'
    : 'bg-gray-200 text-gray-800 hover:bg-gray-300';

  return (
    <button
      type="button"
      className={`${baseStyles} ${variantStyles}`}
      disabled={disabled}
      onClick={onClick}
    >
      {label}
    </button>
  );
};
```

---

## Exemplo: Componente consumindo hook local

```tsx
// components/SearchInput/index.tsx

import { useSearch } from './useSearch';

interface SearchInputProps {
  placeholder?: string;
  onResults: (results: string[]) => void;
}

export const SearchInput = ({ placeholder = 'Buscar...', onResults }: SearchInputProps) => {
  const { query, setQuery } = useSearch(onResults);

  return (
    <input
      type="search"
      value={query}
      onChange={(e) => setQuery(e.target.value)}
      placeholder={placeholder}
      aria-label={placeholder}
    />
  );
};
```

> O hook `useSearch` vive em `components/SearchInput/useSearch.ts` — para exemplos de implementação de hooks, veja [`hooks.md`](./hooks.md).

---

## Exemplo: Componente com types extraídos

Quando os tipos são grandes ou compartilhados com outros componentes, extraia para `types.ts`:

```ts
// components/DataTable/types.ts

export interface Column<T> {
  key: keyof T;
  label: string;
  sortable?: boolean;
  render?: (value: T[keyof T], row: T) => React.ReactNode;
}

export interface DataTableProps<T> {
  columns: Column<T>[];
  data: T[];
  rowKey: keyof T;
  loading?: boolean;
  onRowClick?: (row: T) => void;
  emptyMessage?: string;
}

export interface SortState<T> {
  column: keyof T | null;
  direction: 'asc' | 'desc';
}
```

```tsx
// components/DataTable/index.tsx

import { useState, useMemo } from 'react';
import type { DataTableProps, SortState } from './types';

export const DataTable = <T,>({ columns, data, rowKey, loading, onRowClick, emptyMessage = 'Nenhum dado encontrado' }: DataTableProps<T>) => {
  const [sort, setSort] = useState<SortState<T>>({ column: null, direction: 'asc' });

  const sortedData = useMemo(() => {
    if (!sort.column) return data;

    return [...data].sort((a, b) => {
      const aVal = a[sort.column!];
      const bVal = b[sort.column!];
      const modifier = sort.direction === 'asc' ? 1 : -1;
      return aVal > bVal ? modifier : -modifier;
    });
  }, [data, sort]);

  if (loading) return <div aria-busy="true">Carregando...</div>;
  if (data.length === 0) return <p>{emptyMessage}</p>;

  return (
    <table>
      <thead>
        <tr>
          {columns.map((col) => (
            <th key={String(col.key)} onClick={() => col.sortable && setSort({
              column: col.key,
              direction: sort.column === col.key && sort.direction === 'asc' ? 'desc' : 'asc'
            })}>
              {col.label}
            </th>
          ))}
        </tr>
      </thead>
      <tbody>
        {sortedData.map((row) => (
          <tr key={String(row[rowKey])} onClick={() => onRowClick?.(row)}>
            {columns.map((col) => (
              <td key={String(col.key)}>
                {col.render ? col.render(row[col.key], row) : String(row[col.key])}
              </td>
            ))}
          </tr>
        ))}
      </tbody>
    </table>
  );
};
```

---

## Exemplo: Context/Provider

O Provider é um componente. O hook de consumo (`useAuth`) vive em `src/hooks/` pois é reutilizável globalmente — veja [`hooks.md`](./hooks.md) para o exemplo completo.

```tsx
// components/AuthProvider/index.tsx

import { createContext, useState, useCallback } from 'react';
import type { ReactNode } from 'react';
import type { AuthContextValue, Credentials, User } from '@/hooks/useAuth/types';
import { authService } from '@/services/auth';

export const AuthContext = createContext<AuthContextValue | null>(null);

interface AuthProviderProps {
  children: ReactNode;
}

export const AuthProvider = ({ children }: AuthProviderProps) => {
  const [user, setUser] = useState<User | null>(null);

  const login = useCallback(async (credentials: Credentials) => {
    const response = await authService.login(credentials);
    setUser(response.user);
  }, []);

  const logout = useCallback(() => {
    authService.logout();
    setUser(null);
  }, []);

  const value: AuthContextValue = {
    user,
    login,
    logout,
    isAuthenticated: user !== null,
  };

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
};
```

---

## Anti-patterns (o que evitar)

```tsx
// ❌ Lógica de negócio no JSX
const UserList = ({ users }) => {
  return (
    <ul>
      {users.filter(u => u.active).sort((a, b) => a.name.localeCompare(b.name)).map(u => (
        <li key={u.id}>{u.name}</li>
      ))}
    </ul>
  );
};

// ✅ Lógica extraída
const UserList = ({ users }) => {
  const activeUsers = useMemo(
    () => users.filter(u => u.active).sort((a, b) => a.name.localeCompare(b.name)),
    [users]
  );

  return (
    <ul>
      {activeUsers.map(u => (
        <li key={u.id}>{u.name}</li>
      ))}
    </ul>
  );
};
```

```tsx
// ❌ Prop drilling
const App = () => {
  const [theme, setTheme] = useState('light');
  return <Layout theme={theme} setTheme={setTheme} />;
};

// ✅ Context para valores globais
const App = () => {
  return (
    <ThemeProvider>
      <Layout />
    </ThemeProvider>
  );
};
```
