# Padrão de Modificação

> Arquivo de referência para o fluxo de modificação do `sdd.evolve`.
> Documenta o padrão genérico "Manter", as regras de cascata entre campos,
> e o mapeamento campo → `question-flows.md`.
>
> **Uso:** `sdd.evolve` Fase 4B — para cada campo modificável, aplique o padrão
> abaixo com as opções da seção correspondente em `question-flows.md`.

---

## Campos Fundamentais (não modificáveis)

Alguns campos definem a **fundação** do projeto: trocá-los invalida o código existente em
massa (é uma reescrita, não uma evolução) e/ou determina o conjunto de opções dos demais
campos. O `sdd.evolve` **não modifica** estes campos:

| Camada | Campos fundamentais (imutáveis) |
|--------|---------------------------------|
| Frontend | Framework |
| Backend  | Linguagem · Runtime · Framework |
| Database | Banco principal (engine) |

> Por que isso importa: as opções de ORM, Autenticação e Validação são **escopadas pela
> linguagem**. Com a linguagem fixa, todo `Manter: <atual>` é garantidamente coerente com o
> ecossistema atual — nunca se oferece "Manter: Eloquent" depois de migrar para Node, porque
> a linguagem não pode migrar aqui.

Estes campos **não entram no fluxo "Manter"** da Fase 4B. No resumo do estado (Fase 0) eles
aparecem como referência, marcados `(base — não alterável via evolve)`.

**Se o usuário pedir para trocar um campo fundamental**, recuse com a saída de emergência:

> Trocar este campo (Framework do Frontend, Linguagem/Runtime/Framework do Backend ou o banco
> principal) é uma **reescrita**, não uma evolução — invalida o código e os arquivos da camada.
> O `sdd.evolve` cobre a troca de campos modificáveis (ORM, autenticação, validação, UI, estilização,
> migrations, DevOps, etc.). Para mudar a fundação:
> 1. Refaça via `sdd.setup` (restaurando os placeholders `[ex: ...]` da camada), ou
> 2. Edite manualmente o `spec.md` da camada e os arquivos relacionados.

---

## O Padrão "Manter"

Todo campo **modificável** (não fundamental) segue o mesmo padrão. A opção "Manter" é SEMPRE a primeira.

```
questions: [
  {
    question: "<Nome do campo> — atual: <valor atual>. Deseja alterar?",
    header: "<Header>",
    options: [
      { label: "Manter: <valor atual>", description: "Não alterar" },
      // Opções do `question-flows.md` para este campo,
      // excluindo a que corresponde ao valor atual.
      // + opção "Outra" para valores personalizados.
    ]
  }
]
```

**Regras:**
1. `"Manter: <valor atual>"` é SEMPRE a primeira opção
2. As demais opções vêm da **mesma seção** do `question-flows.md` de onde saiu o valor atual, filtradas para excluir a que corresponde ao valor atual
3. `"Outra"` está sempre disponível para valores personalizados
4. Se o valor atual for `"Não se aplica"`, inclua `"Não se aplica"` como opção normal (não como "Manter")
5. A pergunta é feita **individualmente** — uma chamada `AskUserQuestion` por campo
6. **Coerência de ecossistema** — campos escopados por linguagem (ORM, Autenticação, Validação,
   Migrations) têm duas variantes em `question-flows.md` (`— PHP` e `— TS/JS`). As alternativas saem
   do **mesmo bloco de variante que contém o valor atual** — é o próprio `"Manter: <valor>"` que
   determina a variante, sem precisar consultar a linguagem.

> ⚠️ **Nunca misture ecossistemas numa pergunta.** `Manter: Eloquent` (PHP) só pode aparecer ao lado
> de Doctrine e Cycle ORM (`### ORM — PHP`) — **nunca** Drizzle ou Prisma (`### ORM — TS/JS`). Uma
> pergunta que oferece `Manter: <valor PHP>` junto de opções TS/JS (ou o inverso) é prova de que a
> variante errada foi lida — refaça lendo o bloco que **contém** o valor atual.

**Se o usuário escolher "Manter: ..." em todos os campos de uma camada**, essa camada não sofre
nenhuma alteração — pule a edição de arquivos para ela.

---

## Tabela de Cascata

Quando um campo **modificável** é alterado, pode exigir reavaliação de outros campos.
(Campos fundamentais não mudam no `sdd.evolve`, então não geram cascata.)

| Campo alterado | Efeito cascata |
|---|---|
| **Frontend: Estilização** | Se mudar **de** Tailwind **para** outra, ou vice-versa, reavaliar **UI Library** (Tailwind é pré-requisito para shadcn/ui, Headless UI, DaisyUI). |
| **Backend: ORM** | Se Database estiver ativo, agendar sincronização cross-layer na Fase 5. |
| **Database: ORM** | Se Backend estiver ativo, agendar sincronização cross-layer na Fase 5. |
| **DevOps: Containerização** | Se adicionada ou removida, reavaliar comandos do servidor de dev na Fase 6. |

---

## Mapeamento: Campo → question-flows.md

> A relação campo → seção → `header` é canônica na tabela **"Mapeamento Rápido: Campo → Seção"**
> de `question-flows.md`. Consulte-a para localizar a seção e o `header` de cada campo.
> Esta seção registra apenas a **ordem dos campos por camada** e os **ajustes específicos
> de modificação**.

**Ordem dos campos por camada (modo "Manter"):**

> Campos fundamentais (Framework do Frontend; Linguagem, Runtime e Framework do Backend;
> Banco principal) **não aparecem** abaixo — são imutáveis. Veja "Campos Fundamentais" no topo.

- **Frontend:** Estado global → Estado de servidor → Estilização → Formulários → Roteamento → UI Library
- **Backend:** ORM → Autenticação → Validação
- **Database:** ORM / Query builder → Migrations → Banco de desenvolvimento → Banco de teste
- **DevOps:** CI/CD → Containerização → Hospedagem → Monitoramento → Registry de imagens

**Ajustes específicos de modificação:**

- **Frontend — UI Library:** condicional — só pergunte se o framework (fixo) for React 18 ou
  Next.js 14 **E** a estilização for Tailwind CSS. Senão, pule.
- **Frontend — Roteamento:** use `Roteamento — React 18` ou `Roteamento — Next.js 14` conforme o
  framework (fixo) do projeto.
- **Backend/Database — ORM, Autenticação, Validação, Migrations:** campos **escopados por
  linguagem**. A variante (`— TS/JS` ou `— PHP`) é a que **contém o valor atual** — ex:
  `Manter: Eloquent` ⇒ `### ORM — PHP`. (Coincide com a linguagem fixa do Backend, mas ancorar no
  valor atual dispensa a inferência — ver regra 6 de "O Padrão Manter".)
- **Database — ORM / Query builder:** herdado do Backend se ativo; senão use
  `ORM + Migrations — Sem Backend ativo`.
- **Database — Migrations:** variante que **contém a ferramenta atual** — ex: `Manter: Laravel
  Migrations` ⇒ `### Migrations — PHP` (`### Migrations — TS/JS` para o ecossistema Node).

---

## Casos Especiais

### Tentativa de trocar campo fundamental

Linguagem, Runtime e Framework do Backend, Framework do Frontend e o Banco principal **não são
modificáveis** — veja "Campos Fundamentais (não modificáveis)" no topo deste arquivo e use a
saída de emergência descrita lá.

### Todos os campos mantidos

Se o usuário escolher "Manter: ..." em todos os campos de todas as camadas selecionadas
para modificação, encerre sem modificar arquivos:

> Nenhuma alteração foi solicitada — todos os campos foram mantidos como estão.
> Não há mudanças a aplicar.

### Campo "Não se aplica"

Se o valor atual de um campo for "Não se aplica", as opções seguem o `question-flows.md`
normalmente — a primeira opção é o valor do `question-flows.md` (não "Manter: Não se aplica").
Inclua "Não se aplica" entre as opções se ele aparecer no `question-flows.md`.
