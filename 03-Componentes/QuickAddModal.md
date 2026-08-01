---
aliases: ["QuickAddModal", "Ação rápida"]
tags: [componente]
codigo: ["FinancialControll2.0/src/components/QuickAddModal/index.jsx"]
atualizado: 2026-08-01
---

# QuickAddModal

**`src/components/QuickAddModal`**

Criação de lançamento **sem sair do [[Tela-Dashboard|Dashboard]]**. Aberto pelo
botão ⚡ (`QuickAddButton`) no header.

Mesmo formulário de recorrência e tags do [[ExpenseBox]] — incluindo o
[[TagPicker]].

---

## Reusa os styled components do ExpenseBox

Importa os styled components de `ExpenseBox/styles.js` em vez de duplicar.

> **Consequência:** mexer no visual do modal do ExpenseBox altera este também.
> É o comportamento desejado (consistência), mas precisa ser lembrado ao
> ajustar espaçamento ou largura.

---

## O tipo é escolhido no próprio modal

Diferente do [[ExpenseBox]], onde o tipo vem da página, aqui o usuário escolhe
o bucket. Isso funciona porque `addAccount` faz `{ tipo, ...accountData }` — o
tipo do `accountData` **prevalece** sobre o da página. Ver [[Camada-de-Dados]].

`TIPO_OPTIONS` exclui o **tipo 4** — [[Meta|metas]] não são criáveis aqui.

---

## Atualização imediata do Dashboard

Ao salvar, chama `onCreated(tipo)`, que dispara o `fetchAccounts()` do hook
**daquele tipo**. Os medidores, o donut e o ranking de tags refletem o novo
lançamento na hora — sem reload e sem refazer os outros três fetches.

---

## Não passa `fimAno`

Como o [[ExpenseBox]], um recorrente `MENSAL` criado aqui **termina no ano do
início**. Só a [[Tela-Metas]] permite atravessar o ano. Ver [[Recorrencia]].

---

## Relacionadas
[[Tela-Dashboard]] · [[ExpenseBox]] · [[TagPicker]] · [[Recorrencia]] · [[Camada-de-Dados]]
