---
aliases: ["Recorrência", "Periodicidade"]
tags: [dominio]
codigo: ["FinancialControll2.0/src/utils/recurrence.js"]
atualizado: 2026-08-01
---

# Recorrência

Define **como um [[Lancamento|lançamento]] se espalha no tempo**. É a regra mais
densa do sistema e a que mais gerou bug — quase toda função de
`src/utils/recurrence.js` existe por causa dela.

Três modos, mais o caso legado.

---

## Os três modos

| Modo | Campos que importam | Regra |
|---|---|---|
| `UNICA` | `dia_vencimento`, `data_inicio` = `data_fim` | Ocorre **só** no mês/ano escolhido |
| `MENSAL` | `dia_vencimento`, `data_inicio`, `data_fim` | Todo mês no dia, do início ao fim |
| `ANUAL` | `dia_vencimento`, `data_inicio` (dia+mês), `data_fim` | Uma vez por ano na data, por N anos (**teto: 5**) |

### A pegadinha do `fimAno`

Para `MENSAL`, o formulário pergunta o **mês** final. O **ano** final é
opcional (`fimAno`):

- **Sem `fimAno`** → o fim fica **no mesmo ano do início**. É o comportamento
  das telas de despesa ([[ExpenseBox]], [[QuickAddModal]]).
- **Com `fimAno`** → pode atravessar o ano. **Só a tela de [[Tela-Metas|Metas]]
  passa esse campo**, porque uma meta de 18 meses é normal e uma conta que
  atravessa o ano é exceção.

Isso mantém as telas de despesa simples sem impedir a meta de existir.
`fimAno` é opcional em `validateRecurrence` e `buildRecurrencePayload`
justamente para que nenhuma tela precisasse mudar quando Metas nasceu.

---

## O caso legado

Lançamento com `recorrencia = NULL` foi criado antes da migração
`0002_recorrencia`. Ele **não some e não é migrado** — é interpretado pela regra
antiga: janela de meses consecutivos a partir de `creationMonth`, com duração
`durationMonths` (`qtd_parcelas`).

> **Nunca escreva `if (account.recorrencia === "MENSAL")` direto numa tela.**
> Isso ignora o legado silenciosamente — o lançamento simplesmente some da
> listagem, sem erro. Use os helpers.

`deriveRecurrenceForm(account)` converte legado → estado de formulário na
edição, escolhendo `MENSAL` se `durationMonths > 1`, senão `UNICA`.

---

## As funções e para que servem

Todas em `src/utils/recurrence.js`. Esta tabela é o mapa de qual usar:

| Função | Responde | Trata legado? |
|---|---|---|
| `accountActiveInPeriod(acc, ano, mes)` | "Aparece neste mês?" | ✅ **Use esta** |
| `isActiveInPeriod(acc, ano, mes)` | Idem, mas **só** com recorrência | ❌ |
| `sumActiveInPeriod(accs, ano, mes)` | Soma do que está ativo | ✅ |
| `occurrenceCount(acc)` | Total de ocorrências | ✅ |
| `occurrenceLabel(acc, ano, mes)` | Rótulo `"3/12"` | ✅ |
| `occurrenceCompetencia(acc, ano, mes)` | Qual parcela é esta (`"YYYY-MM"`) | ✅ |
| `isPaidInPeriod(acc, ano, mes)` | Esta parcela está paga? | ✅ |
| `formOccurrenceCount({...})` | Ocorrências a partir do **formulário** | — |
| `validateRecurrence({...})` | Erro de validação ou `null` | — |
| `buildRecurrencePayload({...})` | Campos para o POST/PUT | — |
| `deriveRecurrenceForm(acc)` | Conta → formulário (edição) | ✅ |
| `goalCompetencias(acc)` | Janela de competências da [[Meta]] | — |
| `nextPendingCompetencia(acc)` | Próximo aporte pendente | — |
| `goalProgress(acc)` | Alvo/guardado/% da meta | ✅ |

> **`accountActiveInPeriod` é a porta de entrada.** Ela encapsula
> recorrência + legado e é o que garante que o [[Tela-Dashboard|Dashboard]] some
> exatamente os lançamentos que as telas de despesa mostram. Divergência entre
> dashboard e listagem quase sempre é alguém que reimplementou o filtro.

---

## Filtro parcial (só ano, ou só mês)

Os filtros aceitam ano vazio, mês vazio ou ambos. O comportamento de "só mês"
é o menos óbvio:

- `MENSAL` com só o mês → varre a janela procurando **qualquer** ano em que
  aquele mês caia dentro do período.
- `ANUAL` com só o mês → compara com o mês da `data_inicio`.
- Sem filtro nenhum → mostra tudo.

E a consequência importante: **com "Todos os meses" a competência de um `MENSAL`
é ambígua** — não dá pra saber qual parcela pagar. Ver
[[Competencia-e-Pagamento]].

---

## O que não existe

- **Quinzenal / semanal / diária.** Só os três modos.
- **Dia útil, "todo dia 5 ou próximo útil".** `dia_vencimento` é literal.
- **Fim aberto ("para sempre").** Todo recorrente tem `data_fim`.
- **Valor diferente por ocorrência.** Ver [[Lancamento]].
- **Pular uma ocorrência.** Não há "esta parcela não conta" — só pagar ou não.
- **`ANUAL` além de 5 anos** (`MAX_ANUAL_YEARS`).

---

## Relacionadas
[[Lancamento]] · [[Competencia-e-Pagamento]] · [[Meta]] · [[ExpenseBox]] · [[ADR-005-Filtro-de-periodo-no-frontend]]
