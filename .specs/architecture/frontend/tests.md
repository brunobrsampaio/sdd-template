# Testes — Frontend

> Guia de boas práticas para testes em projetos React.
> Complementa o [`spec.md`](./spec.md) deste mesmo diretório.

---

## Regras Gerais [PADRÃO]

- **Framework:** Jest + React Testing Library (ou Vitest como alternativa)
- **HTTP mocking:** MSW (Mock Service Worker) — proibido `jest.mock` de `fetch`
- **Abordagem:** TDD — testes escritos antes do código de produção. Quando inviável, o teste entra no mesmo commit que a implementação — nunca em PR separado.
- **Cobertura mínima:** 80% de branches na camada de lógica (componentes, hooks e utils)
- **Queries:** sempre semânticas (`getByRole`, `getByLabelText`) — proibido `getByTestId` salvo último recurso
- **O que testar:** comportamento observável pelo usuário, não detalhes de implementação
- **O que não testar:** detalhes internos, estado interno de hooks em isolamento
- **Nome de arquivo de teste:** obrigatório `index.test.ts(x)` dentro da pasta do módulo — proibido `useDebounce.test.ts`, `Button.test.tsx` ou qualquer variação com nome do módulo no arquivo de teste. A razão é dupla: (1) todo módulo com teste deve ser uma pasta (ver "Estrutura de Módulos Internos" no [`spec.md`](./spec.md)), e (2) o ponto de entrada `index` é o contrato público — o teste valida esse contrato.

---

## Hierarquia de Queries (prioridade)

1. **`getByRole`** — acessível, semântico, preferido
2. **`getByLabelText`** — formulários
3. **`getByPlaceholderText`** — inputs sem label visível
4. **`getByText`** — conteúdo visível
5. **`getByDisplayValue`** — valor atual de inputs
6. **`getByTestId`** — último recurso, apenas quando nenhuma outra opção funciona

---

## Exemplo: Teste de componente simples

```tsx
// components/Button/index.test.tsx

import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Button } from '.';

describe('Button', () => {
  it('renders with label', () => {
    render(<Button label="Salvar" onClick={() => {}} />);

    expect(screen.getByRole('button', { name: 'Salvar' })).toBeInTheDocument();
  });

  it('calls onClick when clicked', async () => {
    const user = userEvent.setup();
    const handleClick = jest.fn();

    render(<Button label="Salvar" onClick={handleClick} />);
    await user.click(screen.getByRole('button', { name: 'Salvar' }));

    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('does not call onClick when disabled', async () => {
    const user = userEvent.setup();
    const handleClick = jest.fn();

    render(<Button label="Salvar" onClick={handleClick} disabled />);
    await user.click(screen.getByRole('button', { name: 'Salvar' }));

    expect(handleClick).not.toHaveBeenCalled();
  });
});
```

---

## Exemplo: Teste com estado e interação

```tsx
// components/Counter/index.test.tsx

import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Counter } from '.';

describe('Counter', () => {
  it('starts at initial value', () => {
    render(<Counter initialValue={5} />);

    expect(screen.getByText('5')).toBeInTheDocument();
  });

  it('increments on click', async () => {
    const user = userEvent.setup();
    render(<Counter initialValue={0} />);

    await user.click(screen.getByRole('button', { name: 'Incrementar' }));

    expect(screen.getByText('1')).toBeInTheDocument();
  });

  it('decrements on click', async () => {
    const user = userEvent.setup();
    render(<Counter initialValue={5} />);

    await user.click(screen.getByRole('button', { name: 'Decrementar' }));

    expect(screen.getByText('4')).toBeInTheDocument();
  });

  it('does not go below zero', async () => {
    const user = userEvent.setup();
    render(<Counter initialValue={0} />);

    await user.click(screen.getByRole('button', { name: 'Decrementar' }));

    expect(screen.getByText('0')).toBeInTheDocument();
  });
});
```

---

## Exemplo: Teste com mock de API (MSW)

```ts
// mocks/handlers.ts

import { http, HttpResponse } from 'msw';

export const handlers = [
  http.get('/api/users', () => {
    return HttpResponse.json([
      { id: 1, name: 'João', email: 'joao@example.com' },
      { id: 2, name: 'Maria', email: 'maria@example.com' },
    ]);
  }),

  http.get('/api/users/:id', ({ params }) => {
    return HttpResponse.json({
      id: Number(params.id),
      name: 'João',
      email: 'joao@example.com',
    });
  }),

  http.post('/api/users', async ({ request }) => {
    const body = await request.json();
    return HttpResponse.json({ id: 3, ...body }, { status: 201 });
  }),
];
```

```tsx
// components/UserList/index.test.tsx

import { render, screen, waitFor } from '@testing-library/react';
import { setupServer } from 'msw/node';
import { http, HttpResponse } from 'msw';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { UserList } from '.';

const server = setupServer(
  http.get('/api/users', () => {
    return HttpResponse.json([
      { id: 1, name: 'João', email: 'joao@example.com' },
      { id: 2, name: 'Maria', email: 'maria@example.com' },
    ]);
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

function renderWithProviders(ui: React.ReactElement) {
  const queryClient = new QueryClient({
    defaultOptions: { queries: { retry: false } },
  });

  return render(
    <QueryClientProvider client={queryClient}>
      {ui}
    </QueryClientProvider>
  );
}

describe('UserList', () => {
  it('shows loading state initially', () => {
    renderWithProviders(<UserList />);

    expect(screen.getByText('Carregando...')).toBeInTheDocument();
  });

  it('renders users after loading', async () => {
    renderWithProviders(<UserList />);

    await waitFor(() => {
      expect(screen.getByText('João')).toBeInTheDocument();
      expect(screen.getByText('Maria')).toBeInTheDocument();
    });
  });

  it('shows error message on failure', async () => {
    server.use(
      http.get('/api/users', () => {
        return HttpResponse.json({ message: 'Server error' }, { status: 500 });
      })
    );

    renderWithProviders(<UserList />);

    await waitFor(() => {
      expect(screen.getByText('Erro ao carregar usuários')).toBeInTheDocument();
    });
  });
});
```

---

## Exemplo: Teste de formulário

```tsx
// components/LoginForm/index.test.tsx

import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { LoginForm } from '.';

describe('LoginForm', () => {
  it('renders email and password fields', () => {
    render(<LoginForm onSubmit={() => {}} />);

    expect(screen.getByLabelText('Email')).toBeInTheDocument();
    expect(screen.getByLabelText('Senha')).toBeInTheDocument();
    expect(screen.getByRole('button', { name: 'Entrar' })).toBeInTheDocument();
  });

  it('shows validation errors for empty fields', async () => {
    const user = userEvent.setup();
    render(<LoginForm onSubmit={() => {}} />);

    await user.click(screen.getByRole('button', { name: 'Entrar' }));

    await waitFor(() => {
      expect(screen.getByText('Email é obrigatório')).toBeInTheDocument();
      expect(screen.getByText('Senha é obrigatória')).toBeInTheDocument();
    });
  });

  it('shows error for invalid email', async () => {
    const user = userEvent.setup();
    render(<LoginForm onSubmit={() => {}} />);

    await user.type(screen.getByLabelText('Email'), 'invalido');
    await user.type(screen.getByLabelText('Senha'), '123456');
    await user.click(screen.getByRole('button', { name: 'Entrar' }));

    await waitFor(() => {
      expect(screen.getByText('Email inválido')).toBeInTheDocument();
    });
  });

  it('calls onSubmit with form data when valid', async () => {
    const user = userEvent.setup();
    const handleSubmit = jest.fn();
    render(<LoginForm onSubmit={handleSubmit} />);

    await user.type(screen.getByLabelText('Email'), 'joao@example.com');
    await user.type(screen.getByLabelText('Senha'), 'senha123');
    await user.click(screen.getByRole('button', { name: 'Entrar' }));

    await waitFor(() => {
      expect(handleSubmit).toHaveBeenCalledWith({
        email: 'joao@example.com',
        password: 'senha123',
      });
    });
  });

  it('disables submit button while submitting', async () => {
    const user = userEvent.setup();
    const handleSubmit = jest.fn(() => new Promise((r) => setTimeout(r, 1000)));
    render(<LoginForm onSubmit={handleSubmit} />);

    await user.type(screen.getByLabelText('Email'), 'joao@example.com');
    await user.type(screen.getByLabelText('Senha'), 'senha123');
    await user.click(screen.getByRole('button', { name: 'Entrar' }));

    expect(screen.getByRole('button', { name: 'Entrando...' })).toBeDisabled();
  });
});
```

---

## Exemplo: Teste de hook customizado

```ts
// hooks/useDebounce/index.test.ts

import { renderHook, act } from '@testing-library/react';
import { useDebounce } from '.';

describe('useDebounce', () => {
  beforeEach(() => jest.useFakeTimers());
  afterEach(() => jest.useRealTimers());

  it('returns initial value immediately', () => {
    const { result } = renderHook(() => useDebounce('hello', 500));

    expect(result.current).toBe('hello');
  });

  it('debounces value changes', () => {
    const { result, rerender } = renderHook(
      ({ value, delay }) => useDebounce(value, delay),
      { initialProps: { value: 'hello', delay: 500 } }
    );

    rerender({ value: 'world', delay: 500 });
    expect(result.current).toBe('hello');

    act(() => jest.advanceTimersByTime(500));
    expect(result.current).toBe('world');
  });

  it('resets timer on rapid changes', () => {
    const { result, rerender } = renderHook(
      ({ value, delay }) => useDebounce(value, delay),
      { initialProps: { value: 'a', delay: 300 } }
    );

    rerender({ value: 'ab', delay: 300 });
    act(() => jest.advanceTimersByTime(100));

    rerender({ value: 'abc', delay: 300 });
    act(() => jest.advanceTimersByTime(100));

    expect(result.current).toBe('a');

    act(() => jest.advanceTimersByTime(300));
    expect(result.current).toBe('abc');
  });
});
```

---

## Anti-patterns (o que evitar nos testes)

```tsx
// ❌ Testar detalhes de implementação
it('sets state to true', () => {
  const { result } = renderHook(() => useToggle());
  act(() => result.current.toggle());
  expect(result.current.state).toBe(true);  // isso é detalhe interno
});

// ✅ Testar comportamento observável
it('shows content when toggle is activated', async () => {
  const user = userEvent.setup();
  render(<Accordion title="FAQ" content="Resposta aqui" />);

  await user.click(screen.getByRole('button', { name: 'FAQ' }));

  expect(screen.getByText('Resposta aqui')).toBeVisible();
});
```

```tsx
// ❌ Usar getByTestId quando há alternativa semântica
screen.getByTestId('submit-btn');

// ✅ Usar query semântica
screen.getByRole('button', { name: 'Enviar' });
```

```tsx
// ❌ Mockar fetch diretamente
jest.mock('fetch');

// ✅ Usar MSW para interceptar na camada de rede
server.use(
  http.get('/api/data', () => HttpResponse.json({ items: [] }))
);
```

```ts
// ❌ Arquivo de teste com nome do módulo — fora da pasta
// hooks/useDebounce.test.ts
import { useDebounce } from './useDebounce';

// ❌ Mesmo erro em componentes
// components/Button.test.tsx
import { Button } from './Button';

// ✅ Teste dentro da pasta do módulo, nome fixo index.test.ts
// hooks/useDebounce/index.test.ts
import { useDebounce } from '.';

// ✅ Componente também segue o mesmo padrão
// components/Button/index.test.tsx
import { Button } from '.';
```

> **Regra:** se o módulo tem teste, ele **obrigatoriamente** é uma pasta com `index.ts(x)` + `index.test.ts(x)`.
> Arquivo de teste com nome do módulo (`useDebounce.test.ts`, `Button.test.tsx`) é proibido — não existe
> cenário em que um módulo testado justifique ser arquivo único fora de pasta.
