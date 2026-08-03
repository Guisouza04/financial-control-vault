---
aliases: ["Contas", "Investimentos", "Opcionais", "Telas de despesa"]
tags: [tela]
rota: "/Contas · /Investments · /Optional"
codigo: ["FinancialControll2.0/src/pages/Contas/index.jsx"]
atualizado: 2026-08-02
---

# Telas · Contas · Investimentos · Opcionais

**Rotas:** `/Contas` · `/Investments` · `/Optional`

As três são **a mesma tela**. Cada página é um wrapper de três linhas que monta
o [[ExpenseBox]] com um [[Tipo|tipo]] diferente:

```jsx
<ExpenseBox tipo={1} />   // Contas
<ExpenseBox tipo={2} />   // Investimentos
<ExpenseBox tipo={3} />   // Opcionais
```

> **Toda a funcionalidade está em [[ExpenseBox]].** Se você quer mudar
> comportamento de listagem, filtro, modal ou pagamento, é lá — mexer numa das
> páginas muda uma só e cria divergência.

**Metas (tipo 4) não usa este componente** — ver [[Tela-Metas]] e [[Meta]].

---

## O que o usuário consegue fazer

| Ação | Como |
|---|---|
| **Pular para outra seção** (Investimentos, Opcionais, Metas) | Barra de abas na linha do título — ver [[FinanceTabs]]. Não precisa voltar ao hub |
| Ver os lançamentos do tipo no período | Tabela, default mês/ano atuais |
| Filtrar | Ano (input) + Mês (Select) + Status (Todas/Pagas/Pendentes) + Tag |
| Voltar ao mês corrente | Botão "Mês Atual" (vira "Data Atual" se o ano difere) |
| Criar lançamento | Botão → modal com nome, valor (máscara BRL), recorrência, tags |
| Editar | ✏️ na linha → mesmo modal, pré-preenchido |
| Excluir | Com confirmação |
| Marcar parcela como paga | Toggle, com confirmação |
| Categorizar | [[TagPicker]] no modal; chips coloridos na coluna Nome |

---

## Colunas

**Nome** (+ chips de tag, + 💳 se `na_fatura`) · **Valor** (`R$ 1.234,56`) ·
**Parcela** (`3/12`) · **Status** (verde "Paga" / vermelho "Pendente") ·
**Ações**

---

## Comportamentos que costumam surpreender

### Paginação é dinâmica
Itens por página são **calculados pela altura disponível**, via `ResizeObserver`
no `TableWrapper`. A tabela ocupa a altura da tela e enche de linhas. Não há
número fixo — redimensionar a janela muda quantos itens aparecem.

### O toggle de pagamento bloqueia em "Todos os meses"
Um lançamento `MENSAL` sem mês selecionado tem competência **ambígua** — não dá
para saber qual parcela pagar. O toggle bloqueia e pede para escolher um mês.
Ver [[Competencia-e-Pagamento]].

> A [[Tela-Metas]] **não** bloqueia nessa situação — ela mira o próximo aporte
> pendente. Comportamentos diferentes, de propósito.

### Recorrentes aparecem em todos os meses da janela
Uma conta mensal é **uma** linha no banco. Ela aparece em cada mês do período,
com a coluna Parcela indicando `n/total`. Não são registros duplicados.

### Lançamentos legados continuam aparecendo
Registros sem `recorrencia` seguem a regra antiga (`creationMonth` +
`durationMonths`). O filtro trata os dois casos — ver [[Recorrencia]].

---

## O que estas telas NÃO fazem

- **Não criam metas** (tipo 4) — `TIPO_OPTIONS` exclui o 4.
- **Não movem lançamento entre tipos** pela tabela (só editando).
- **Não têm ações em massa** — sem seleção múltipla, sem exclusão em lote.
- **Não exportam** (CSV/PDF).
- **Não ordenam por coluna** — a ordem é a que vem da API.
- **Não buscam por texto** — só os filtros de ano/mês/status/tag.

---

## Relacionadas
[[ExpenseBox]] · [[Tipo]] · [[Recorrencia]] · [[Competencia-e-Pagamento]] · [[Tag]] · [[Tela-Metas]]
