# Guia de Arquitetura — Banco de Dados

> Regras específicas para modelagem, migrations e acesso a dados.
> Complementa a constitution — não a substitui.
> Seções marcadas com [PROJETO] devem ser ajustadas por projeto.
> Seções marcadas com [PADRÃO] refletem boas práticas gerais.
> Remover esse bloco ao iniciar um novo projeto. O arquivo deve começar a partir da seção `Stack`

---

## Stack [PROJETO]

- **Banco principal:** [ex: PostgreSQL 16]
- **ORM / Query builder:** [ex: Drizzle ORM]
- **Migrations:** [ex: Drizzle Kit — proibido alterar schema sem migration]
- **Banco de desenvolvimento:** [ex: Docker local com pg]
- **Banco de teste:** [ex: instância isolada via Docker Compose]

---

## Nomenclatura [PADRÃO]

- Tabelas: `snake_case`, plural (`user_profiles`, `order_items`)
- Colunas: `snake_case` (`created_at`, `user_id`)
- Chaves primárias: sempre `id` do tipo UUID (`id UUID PRIMARY KEY DEFAULT gen_random_uuid()`)
- Chaves estrangeiras: `<tabela_referenciada_singular>_id` (`user_id`, `order_id`)
- Índices: `idx_<tabela>_<coluna(s)>` (`idx_orders_user_id`)
- Constraints: `chk_<tabela>_<descricao>` (`chk_orders_status`)

---

## Modelagem [PADRÃO]

- Toda tabela tem: `id`, `created_at`, `updated_at`
- Proibido deletar registros fisicamente sem análise — prefira soft delete com `deleted_at`
- Relacionamentos N:N têm tabela de junção com nome `<tabela_a>_<tabela_b>` (`user_roles`)
- Proibido armazenar dados derivados que podem ser calculados — salvo performance justificada
- Enums de banco apenas para valores verdadeiramente fixos — use tabela de lookup para o resto

---

## Migrations [PADRÃO]

- Toda alteração de schema passa por migration — proibido alterar banco diretamente
- Migrations são irreversíveis por padrão — `down` opcional, documentar quando presente
- Nome da migration descreve a mudança: `add_deleted_at_to_orders`, `create_user_roles_table`
- Migrations não contêm lógica de negócio — apenas DDL e DML simples
- Testar migration em ambiente de desenvolvimento antes de aplicar em staging

---

## Acesso a Dados [PADRÃO]

- Todo acesso ao banco passa pela camada de repositório — proibido query direta em serviços
- Repositórios expõem métodos com nomes de domínio (`findByEmail`, `listActiveOrders`)
- Proibido `SELECT *` — sempre listar colunas explicitamente
- Queries com `JOIN` complexo ou subquery são documentadas com comentário explicativo
- Transações usadas sempre que múltiplas operações precisam ser atômicas

---

## Performance [PADRÃO]

- Índice criado para toda coluna usada frequentemente em `WHERE` ou `JOIN`
- Proibido query dentro de loop — use batch ou `IN`
- Queries lentas (> 100ms) são analisadas com `EXPLAIN ANALYZE` antes de ir para produção
- Paginação obrigatória em queries que podem retornar conjuntos grandes (cursor ou offset+limit)

---

## Segurança [PADRÃO]

- Proibido concatenar strings para montar queries — use prepared statements / parâmetros
- Credenciais do banco apenas via variável de ambiente — proibido hardcode
- Usuário do banco em produção tem apenas as permissões necessárias — proibido usar superuser
