---
aliases: ["ConfirmDialog", "useConfirm", "Confirmação"]
tags: [componente]
codigo: ["FinancialControll2.0/src/components/ConfirmDialog/index.jsx", "FinancialControll2.0/src/context/confirm.js"]
atualizado: 2026-08-01
---

# ConfirmDialog

**`src/components/ConfirmDialog`** + **`src/context/confirm.js`**

Substituto de `window.confirm()` com o tema do app. A API é uma **Promise**:

```js
const confirm = useConfirm();

const ok = await confirm({ /* opções */ });
if (!ok) return;
// … prossegue com a ação destrutiva
```

`confirm(options)` → `Promise<boolean>`.

> **Por que não `window.confirm`:** o nativo não é estilizável, e — mais
> importante — **bloqueia a thread**, o que trava animações e, em automação de
> browser, congela a sessão inteira.

---

## Onde é usado

Toda ação destrutiva ou de difícil reversão:
- Excluir lançamento ([[ExpenseBox]])
- Toggle de pagamento
- Excluir [[Tag|tag]] ([[TagPicker]]) — que é **global**, afeta todos os
  lançamentos

---

## Mesmo padrão do [[Toast]]

Context e hook em `src/context/confirm.js`, provider no componente. Separados
para não quebrar o **Fast Refresh** do Vite.

`useConfirm()` **lança erro** se usado fora do `<ConfirmProvider>` — falha alta
e clara em vez de `null` silencioso.

> Se você criar uma página nova com ação destrutiva, ela precisa estar dentro
> do provider (montado no `main.jsx`).

---

## Relacionadas
[[Toast]] · [[ExpenseBox]] · [[TagPicker]]
