---
aliases: ["Antes de implementar", "Checklist"]
tags: [guia]
atualizado: 2026-08-01
---

# Guia · Antes de implementar qualquer coisa

O checklist que evita retrabalho neste projeto. Leva dois minutos.

---

## 1 · Identifique o que você está tocando

| Se a mudança envolve… | Leia antes |
|---|---|
| Repetição no tempo | [[Recorrencia]] |
| "Marcar como pago" | [[Competencia-e-Pagamento]] → [[ADR-002-Pagamento-por-competencia]] |
| Metas / progresso | [[Meta]] → [[ADR-003-Metas-derivadas]] |
| Soma, total, dashboard | [[Tipo]] → [[Salario]] → [[ADR-005-Filtro-de-periodo-no-frontend]] |
| Cartão / importação | [[Fatura-de-Cartao]] → [[ADR-001-naFatura-e-marcador]] |
| Categorização | [[Tag]] |
| Campo novo no lançamento | [[Guia-Novo-Campo-no-Lancamento]] |
| Listagem / filtro / modal de despesa | [[ExpenseBox]] |
| Data ou dropdown | [[ADR-004-Controles-customizados]] |

---

## 2 · As cinco armadilhas do projeto

### ⚠️ Registros legados existem e não somem
Lançamentos com `recorrencia = NULL` são tratados por uma regra **diferente**.
Escrever `if (account.recorrencia === "MENSAL")` faz o legado **sumir da tela
sem erro nenhum**.

> **Use `accountActiveInPeriod`**, não reimplemente o filtro.

### ⚠️ Competência pode ser `null`
Um `MENSAL` sem mês no filtro não identifica parcela. Toda UI que marca
pagamento **precisa** tratar isso. [[ExpenseBox]] bloqueia;
[[Tela-Metas]] mira o próximo pendente.

### ⚠️ `transformAccount` é o gargalo do shape
Campo que não passa por lá chega no backend, é salvo, e **some na volta** —
silenciosamente. Ver [[Camada-de-Dados]].

### ⚠️ `tag_ids` substitui, não faz merge
Omitir **remove todas** as tags. Ver [[Tag]].

### ⚠️ Dashboard e listagem precisam concordar
Os dois somam via `accountActiveInPeriod`. Se divergirem, alguém reimplementou
o filtro. Ver [[ADR-005-Filtro-de-periodo-no-frontend]].

---

## 3 · Tem ADR sobre isso?

Antes de "consertar" algo que parece estranho, confira
[[05-Decisoes/ADR-Index|o índice de ADRs]]. As decisões mais
contra-intuitivas do sistema estão lá — e várias são exatamente o tipo de coisa
que alguém "corrigiria" por engano.

Casos clássicos:
- `na_fatura` **não** exclui do total ([[ADR-001-naFatura-e-marcador|001]])
- Metas **não** têm tabela ([[ADR-003-Metas-derivadas|003]])
- O filtro **é** client-side de propósito ([[ADR-005-Filtro-de-periodo-no-frontend|005]])

---

## 4 · Ao mexer no backend

- **Migração aditiva**, coluna nullable, fallback em código. Ver [[Migracoes]]
- Erro no formato `{ "error": "mensagem" }`, senão a UI cai no genérico
- Endpoint novo → **documente em [[Endpoints]]**

---

## 5 · Verificação

Não há teste automatizado. **Verificar é abrir o app.**

| Mudou | Cheque também |
|---|---|
| Regra de período | [[Tela-Dashboard]] **e** [[Tela-Despesas]] — devem bater |
| Pagamento | Com mês selecionado **e** em "Todos os meses" |
| Recorrência | Um registro legado, se houver |
| Metas | Editar o início e conferir se o progresso não inflou |
| Modal do ExpenseBox | O [[QuickAddModal]] (reusa os styled components) |

---

## 6 · Ao terminar

1. **Atualize a nota do vault** correspondente — é parte do "pronto"
2. Se houve trade-off, **escreva um ADR**
3. Atualize o `CLAUDE.md` dos repos afetados (regra existente do projeto)

---

## Relacionadas
[[Mapa-do-Sistema]] · [[Guia-Novo-Campo-no-Lancamento]] · [[Convencoes]] · [[Ambiente-de-Dev]]
