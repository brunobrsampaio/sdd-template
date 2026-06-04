# Hooks — Frontend

> Guia de boas práticas para criação de hooks customizados React.
> Complementa o `spec.md` deste mesmo diretório.

---

## Regras Gerais [PADRÃO]

- Hooks são sempre arrow functions — `export const useHook = () => {}`
- Prefixo `use` obrigatório — sem exceções
- Hooks não devem conter JSX — se renderiza algo, é um componente
- Hooks globais vivem em `src/hooks/` — hooks locais (usados por um único componente) vivem na pasta do componente

> Para a regra de arquivo único vs pasta, veja "Estrutura de Módulos Internos" no [`spec.md`](./spec.md).

---

## Exemplo: Hook simples (arquivo único)

```ts
// hooks/useDebounce.ts

import { useState, useEffect } from 'react';

export const useDebounce = <T>(value: T, delay: number): T => {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
};
```

---

## Exemplo: Hook simples — toggle

```ts
// hooks/useToggle.ts

import { useState, useCallback } from 'react';

interface UseToggleReturn {
  isOpen: boolean;
  open: () => void;
  close: () => void;
  toggle: () => void;
}

export const useToggle = (initialState = false): UseToggleReturn => {
  const [isOpen, setIsOpen] = useState(initialState);

  const open = useCallback(() => setIsOpen(true), []);
  const close = useCallback(() => setIsOpen(false), []);
  const toggle = useCallback(() => setIsOpen((prev) => !prev), []);

  return { isOpen, open, close, toggle };
};
```

---

## Exemplo: Hook com pasta (tipos e testes separados)

```ts
// hooks/useAuth/types.ts

export interface User {
  id: string;
  name: string;
  email: string;
  role: 'admin' | 'user';
}

export interface Credentials {
  email: string;
  password: string;
}

export interface AuthContextValue {
  user: User | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  login: (credentials: Credentials) => Promise<void>;
  logout: () => void;
}
```

```ts
// hooks/useAuth/index.ts

import { useContext } from 'react';
import { AuthContext } from '@/components/AuthProvider';
import type { AuthContextValue } from './types';

export const useAuth = (): AuthContextValue => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
};
```

```ts
// hooks/useAuth/index.test.ts

import { renderHook } from '@testing-library/react';
import { useAuth } from '.';
import { AuthProvider } from '@/components/AuthProvider';

describe('useAuth', () => {
  it('throws when used outside AuthProvider', () => {
    expect(() => renderHook(() => useAuth())).toThrow(
      'useAuth must be used within an AuthProvider'
    );
  });

  it('returns context value when inside AuthProvider', () => {
    const { result } = renderHook(() => useAuth(), {
      wrapper: AuthProvider,
    });

    expect(result.current.user).toBeNull();
    expect(result.current.isAuthenticated).toBe(false);
  });
});
```

---

## Exemplo: Hook com dependência de outro hook

```ts
// hooks/useLocalStorage.ts

import { useState, useCallback } from 'react';

export const useLocalStorage = <T>(key: string, initialValue: T): [T, (value: T) => void, () => void] => {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });

  const setValue = useCallback((value: T) => {
    setStoredValue(value);
    window.localStorage.setItem(key, JSON.stringify(value));
  }, [key]);

  const removeValue = useCallback(() => {
    setStoredValue(initialValue);
    window.localStorage.removeItem(key);
  }, [key, initialValue]);

  return [storedValue, setValue, removeValue];
};
```

---

## Exemplo: Hook para fetch com TanStack Query

```ts
// hooks/useUsers.ts

import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { userService } from '@/services/user';
import type { User } from '@/types/user';

export const useUsers = () => {
  return useQuery<User[]>({
    queryKey: ['users'],
    queryFn: userService.getAll,
  });
};

export const useCreateUser = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: userService.create,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
};
```

---

## Anti-patterns (o que evitar)

```ts
// ❌ Hook que retorna JSX
export const useModal = () => {
  const [isOpen, setIsOpen] = useState(false);
  const Modal = () => <div className="modal">...</div>;
  return { isOpen, Modal };
};

// ✅ Hook retorna estado/ações — componente renderiza separadamente
export const useModal = () => {
  const [isOpen, setIsOpen] = useState(false);
  const open = () => setIsOpen(true);
  const close = () => setIsOpen(false);
  return { isOpen, open, close };
};
```

```ts
// ❌ Hook sem prefixo use
export const fetchUsers = () => {
  const [users, setUsers] = useState([]);
  // ...
};

// ✅ Com prefixo
export const useFetchUsers = () => {
  const [users, setUsers] = useState([]);
  // ...
};
```

```ts
// ❌ Lógica de negócio complexa misturada com estado
export const useCart = () => {
  const [items, setItems] = useState([]);

  const addItem = (item) => {
    const existingIndex = items.findIndex(i => i.id === item.id);
    if (existingIndex >= 0) {
      const updated = [...items];
      updated[existingIndex].quantity += 1;
      const discount = updated[existingIndex].quantity > 3 ? 0.1 : 0;
      updated[existingIndex].price = item.basePrice * (1 - discount);
      setItems(updated);
    } else {
      setItems([...items, { ...item, quantity: 1 }]);
    }
  };

  return { items, addItem };
};

// ✅ Lógica de negócio extraída para util
import { mergeCartItem } from '@/utils/cart';

export const useCart = () => {
  const [items, setItems] = useState([]);

  const addItem = (item) => {
    setItems((prev) => mergeCartItem(prev, item));
  };

  return { items, addItem };
};
```
