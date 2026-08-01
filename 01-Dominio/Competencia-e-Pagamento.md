---
aliases: ["Competência", "Pagamento por competência", "Status por parcela"]
tags: [dominio]
codigo: ["FinancialControllBackend/app/models/pagamento.py", "FinancialControll2.0/src/utils/recurrence.js"]
atualizado: 2026-08-01
---

# Competência e Pagamento

O problema que esta regra resolve, em uma frase:

> **Um lançamento recorrente é UMA linha no banco que aparece em VÁRIOS meses.
> Logo, "está pago" não pode ser um campo da linha.**

Se fosse, marcar a conta de luz de agosto como paga marcaria também setembro,
outubro e todos os outros. Por isso o pagamento é rastreado **por competência**.

---

## Competência

Uma string `"YYYY-MM"` que identifica **uma parcela específica** de um
lançamento. É a chave de tudo aqui.

**Modelo no banco** (`pagamentos`):

| Coluna | |
|---|---|
| `lancamento_id` | FK, `ondelete=CASCADE` |
| `competencia` | `String(7)` — `"2026-08"` |
| `dt_create` | quando foi marcada |

Com `UniqueConstraint(lancamento_id, competencia)`.

> **Pendente é a AUSÊNCIA de linha.** Não existe registro com `pago = false`.
> Marcar → insere; desmarcar → remove. Isso mantém a tabela pequena e torna
> impossível o estado inconsistente "duas linhas discordando".

---

## Contrato da API

`GET` devolve, em cada lançamento:
```json
"pagamentos": ["2026-08", "2026-10"]
```

`PUT /financas/payment-status/{id}`:
```json
{ "competencia": "2026-08", "conta_paga": "S" }
```

---

## Derivando a competência da parcela exibida

`occurrenceCompetencia(account, filterYear, filterMonth)` responde *"qual
parcela é essa que estou vendo?"*:

| Recorrência | Competência |
|---|---|
| `UNICA` | Sempre o mês/ano da própria ocorrência (ignora o filtro) |
| `ANUAL` | Mês do início + ano **do filtro** (ou do início) |
| `MENSAL` | Ano + mês **do filtro** |
| Legado | Ano + mês **do filtro** |
| `MENSAL`/legado **sem mês no filtro** | **`null`** ⚠️ |

### O `null` e o que ele obriga

Com o filtro em "Todos os meses", um lançamento `MENSAL` aparece na lista mas
**não dá para saber qual parcela o usuário quer pagar**. A função devolve
`null`, e a UI é obrigada a lidar com isso:

- **[[ExpenseBox]]:** o toggle de pagamento **bloqueia** e pede para selecionar
  um mês.
- **[[Tela-Metas|Metas]]:** não bloqueia — cai em `nextPendingCompetencia()`,
  o primeiro mês da janela ainda não aportado. Ver [[Meta]].

Duas telas, dois comportamentos, mesma ambiguidade — porque em despesas o
usuário quer pagar *um mês específico*, e numa meta ele quer *avançar*.

---

## Compatibilidade com o legado

`isPaidInPeriod` só usa `pagamentos` se ele for um array. Se vier ausente
(`null`) — backend antigo, registro anterior à migração `0003_pagamentos` — cai
no campo único `conta_paga`, que é tudo-ou-nada.

**Não remova esse fallback** sem antes garantir que o backend sempre envia
`pagamentos` e que não há registro sem ele.

---

## Armadilhas conhecidas

1. **Pagamento órfão.** Editar o período de um recorrente **não limpa**
   `pagamentos`. Uma competência que caiu fora da nova janela continua no banco.
   Em despesas isso é inofensivo (a parcela nem é exibida); em [[Meta|metas]]
   isso inflava o progresso — e foi um bug real. A correção foi
   `goalCompetencias`, que filtra os pagamentos pela janela atual.

2. **Marcar em "Todos os meses"** — resolvido pelo bloqueio acima. Se você
   adicionar uma nova tela que marque pagamento, **precisa** tratar o `null`.

3. **`conta_paga` continua no modelo** e é gravado em alguns caminhos. Ele
   **não é** a fonte de verdade. Não escreva lógica nova em cima dele.

---

## Relacionadas
[[Recorrencia]] · [[Lancamento]] · [[Meta]] · [[ADR-002-Pagamento-por-competencia]] · [[ExpenseBox]]
