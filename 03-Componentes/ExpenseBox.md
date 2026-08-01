---
aliases: ["ExpenseBox"]
tags: [componente]
codigo: ["FinancialControll2.0/src/components/ExpenseBox/index.jsx"]
atualizado: 2026-08-01
---

# ExpenseBox

**`src/components/ExpenseBox/index.jsx`**

O componente mais importante do frontend. **Três telas inteiras são ele** —
[[Tela-Despesas|Contas, Investimentos e Opcionais]] diferem apenas na prop
`tipo`.

```jsx
<ExpenseBox tipo={1 | 2 | 3} />
```

> **Mudança de comportamento de listagem vai aqui.** Alterar uma das páginas
> muda uma só e cria divergência silenciosa entre as três.

---

## Responsabilidades

| Área | O que faz |
|---|---|
| **Dados** | Consome [[Camada-de-Dados\|useAccounts(tipo, ano, mes)]] |
| **Filtros** | Ano (input) · Mês (Select) · Status · Tag |
| **Tabela** | Nome (+chips +💳) · Valor · Parcela · Status · Ações |
| **Paginação** | Dinâmica, por altura disponível |
| **Modal** | Criar **e** editar — o mesmo |
| **Pagamento** | Toggle por competência, com confirmação |

---

## Paginação dinâmica

Itens por página são calculados pela **altura disponível** da tabela, via
`ResizeObserver` no `TableWrapper`. A tabela ocupa a altura da tela e enche de
linhas.

Layout: `flex` do `.frame` até o corpo, cabeçalho `sticky`, scroll no
`TableWrapper`.

> Não há constante de "itens por página". Redimensionar a janela muda a
> paginação. Se você mexer na altura de linha ou no header, a conta muda junto.

---

## Filtro de período — use os helpers

- Com `recorrencia` → `isActiveInPeriod` / `occurrenceLabel`
- **Legado** (sem `recorrencia`) → janela por `creationMonth` + `durationMonths`
- Sem filtro → mostra tudo

> **Nunca reimplemente essa regra.** `accountActiveInPeriod` de
> `src/utils/recurrence.js` encapsula os dois casos, e é o que garante que o
> [[Tela-Dashboard|Dashboard]] some exatamente o que esta tela lista.
> Ver [[Recorrencia]].

---

## O bloqueio do toggle em "Todos os meses"

Um `MENSAL` sem mês selecionado tem competência **ambígua**
(`occurrenceCompetencia` devolve `null`). O toggle **bloqueia** e pede para
selecionar um mês.

A [[Tela-Metas]] trata a mesma ambiguidade de forma **oposta** (mira o próximo
aporte pendente) — porque lá o usuário quer avançar, não quitar um mês
específico. Ver [[Competencia-e-Pagamento]].

---

## Modal único para criar e editar

Nome · Valor (máscara BRL) · Recorrência (Única/Mensal/Anual + campos
condicionais) · [[TagPicker]].

- Editar (✏️) pré-preenche via `deriveRecurrenceForm(account)`, que também
  converte registros legados
- **O início é preservado** na edição — dá para mudar o fim, converter
  única↔mensal, mas não reescrever a data de origem
- **Não passa `fimAno`** → o fim fica no ano do início. Só [[Tela-Metas]] passa.
  Ver [[Recorrencia]]

---

## `TIPO_OPTIONS` exclui o tipo 4

[[Meta|Metas]] não são criáveis aqui. É deliberado — criar meta é ação da
página `/metas`.

---

## Reuso dos styled components

O [[QuickAddModal]] **importa os styled components daqui** em vez de duplicar.
Mexer nos estilos do modal afeta os dois.

---

## Relacionadas
[[Tela-Despesas]] · [[Recorrencia]] · [[Competencia-e-Pagamento]] · [[Tag]] · [[TagPicker]] · [[QuickAddModal]] · [[Camada-de-Dados]]
