---
aliases: ["Novo campo no lançamento", "Adicionar campo"]
tags: [guia]
atualizado: 2026-08-01
---

# Guia · Adicionar um campo no lançamento

O caminho completo, ponta a ponta. **Pular um passo dá bug silencioso** — o
mais comum é o campo salvar e sumir na volta.

Use a migração `0005_data_compra` como referência: ela é um exemplo completo e
pequeno.

---

## Backend

### 1 · Migração (aditiva, nullable)
`FinancialControllBackend/alembic/versions/000N_seu_campo.py`

```python
op.add_column("lancamentos", sa.Column("seu_campo", sa.String(), nullable=True))
```

> **Sempre nullable.** Registros existentes não serão migrados — o padrão do
> projeto é compatibilidade em código, não backfill. Ver [[Migracoes]].

### 2 · Model
`app/models/lancamento.py` — e **comente o porquê**, não o quê:

```python
# Por que este campo existe e o que ele NÃO é.
seu_campo: Mapped[str | None] = mapped_column(String(64), nullable=True)
```

### 3 · Schemas
`app/schemas/financas.py` — nos **três**:
- `LancamentoIn` (create) — com default, para não quebrar clientes antigos
- `LancamentoUpdate` (update)
- `LancamentoOut` (resposta) ← **o mais esquecido**

### 4 · Endpoints
`app/api/financas.py` — gravar no `create` e no `update`.

---

## Frontend

### 5 · `transformAccount` ⚠️ **o passo crítico**
`src/services/financeService.js`

```js
seuCampo: item.seu_campo,
```

> **Se você esquecer aqui, o campo é salvo no banco e nunca aparece na UI** —
> sem erro, sem aviso. É o bug mais caro deste caminho.

### 6 · Payload de envio
No mesmo arquivo, no `createAccount`/`updateAccount` — converter
`camelCase` → `snake_case`.

### 7 · Formulário
[[ExpenseBox]] (modal) e, se fizer sentido, [[QuickAddModal]] — que **reusa os
styled components** do ExpenseBox.

### 8 · Edição
Se o campo entra no formulário de recorrência, `deriveRecurrenceForm`
(`src/utils/recurrence.js`) precisa devolvê-lo — **incluindo o caso legado**.

### 9 · Exibição
Tabela do [[ExpenseBox]], e/ou [[Tela-Dashboard|Dashboard]] se afetar somas.

---

## Checklist

- [ ] Migração aditiva, coluna nullable
- [ ] Model + comentário do **porquê**
- [ ] Schema **In**, **Update** e **Out**
- [ ] `create` e `update` gravam
- [ ] **`transformAccount` mapeia** ⚠️
- [ ] Payload de envio converte para `snake_case`
- [ ] Formulário (ExpenseBox + QuickAddModal)
- [ ] `deriveRecurrenceForm`, se aplicável
- [ ] Exibição
- [ ] **Legado tratado** — o que acontece quando o campo é `NULL`?
- [ ] Nota atualizada em [[Lancamento]] e [[Endpoints]]
- [ ] ADR, se houve trade-off

---

## Se o campo afeta soma ou período

Aí não é só um campo — mexe na regra central.
**Leia [[ADR-005-Filtro-de-periodo-no-frontend]] antes**, e garanta que
[[Tela-Dashboard]] e [[Tela-Despesas]] continuem concordando.

---

## Relacionadas
[[Lancamento]] · [[Camada-de-Dados]] · [[Migracoes]] · [[Guia-Antes-de-Implementar]]
