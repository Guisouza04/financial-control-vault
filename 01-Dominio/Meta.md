---
aliases: ["Meta", "Metas", "Aporte"]
tags: [dominio]
codigo: ["FinancialControll2.0/src/pages/Metas/index.jsx", "FinancialControll2.0/src/utils/recurrence.js"]
atualizado: 2026-08-01
---

# Meta

O 4º balde da divisão 60/20/10/10 ([[Tipo]] = 4). Uma meta é um **aporte
recorrente para um objetivo** — "R$ 500/mês por 12 meses para a viagem" — não
um boleto a vencer.

Essa diferença de natureza é o que justifica tela própria com cards de
progresso, em vez da tabela do [[ExpenseBox]].

---

## Alvo e progresso são DERIVADOS

**Não houve migração para metas.** Não existe tabela `metas`, nem coluna
`valor_alvo`, nem `valor_acumulado`. O modelo [[Lancamento]] já tinha tudo:

| Conceito | Deriva de | Onde |
|---|---|---|
| **Alvo** | `value × occurrenceCount(account)` | `goalProgress()` |
| **Guardado** | `value × aportes dentro da janela` | `goalProgress()` |
| **Registrar aporte** | `togglePaymentStatus(id, competencia, pago)` | `useAccounts` |
| **Concluída** | `aportesPagos >= totalAportes` | `goalProgress().concluida` |

> **"Parcela paga" lê-se "aporte feito".** É literalmente o mesmo mecanismo de
> [[Competencia-e-Pagamento]], com outro nome no vocabulário da tela.

Ver [[ADR-003-Metas-derivadas]].

---

## A janela da meta — e o bug que ela corrigiu

`goalCompetencias(account)` lista as competências que a meta cobre.
`goalProgress` conta **apenas** os `pagamentos` que caem nessa janela.

Isso não é preciosismo. Foi um **bug real encontrado em uso**:

> Uma meta começava em julho e recebeu um aporte (`pagamentos: ["2026-07"]`).
> O usuário editou o início para setembro. O aporte de julho ficou **órfão** —
> fora da nova janela — mas continuava contando como progresso **para sempre**.

Sem o filtro pela janela, o progresso mente. Se você mexer em `goalProgress`,
esse é o comportamento a preservar.

---

## O botão de aporte segue a META, não o filtro

O Ano/Mês no topo da tela de Metas **não filtra a lista** — ele só define **de
qual competência é o aporte**. A lista mostra sempre todas as metas.

Motivo: uma meta atravessa meses. Se ela sumisse ao filtrar por um mês fora da
janela, o usuário perderia de vista o próprio objetivo.

`competenciaDoAporte` decide assim:

1. Usa o **mês do filtro**, *se* ele fizer parte da janela da meta.
2. Senão, cai em `nextPendingCompetencia()` — o primeiro mês da janela ainda
   não aportado.

Assim, mover o início de uma meta para setembro faz o botão oferecer
**"Registrar aporte de Setembro/2026"**, em vez de travar dizendo que o filtro
está fora da meta.

> **O rótulo traz mês E ano** (`Setembro/2026`, não `Setembro`) — a meta
> atravessa anos e o mês sozinho seria ambíguo.

**Consequência:** com "Todos os meses" o botão de aporte **continua
funcionando** (mira o próximo pendente). É o oposto do [[ExpenseBox]], que
bloqueia. Ver [[Competencia-e-Pagamento]].

---

## Meta pode atravessar o ano

O modal de meta escolhe **ano e mês** de término (`fimAno`) — diferente das
despesas, cujo fim é sempre no ano do início. `formOccurrenceCount` calcula as
ocorrências e alimenta `qtd_parcelas` no payload.

`fimAno` é opcional em `validateRecurrence`/`buildRecurrencePayload`
exatamente para que [[ExpenseBox]] e [[QuickAddModal]] não precisassem mudar.
Ver [[Recorrencia]].

---

## Metas não têm tags

Decisão de produto: o modal de meta não monta [[TagPicker]]. Como `tag_ids` tem
default `[]` e o update **substitui** o conjunto, omitir o campo já deixa a meta
sem tags — sem código extra. Ver [[Tag]].

---

## Metas não são criáveis fora da tela de Metas

`TIPO_OPTIONS` (usado por [[ExpenseBox]] e [[QuickAddModal]]) **exclui o tipo 4**.
Criar meta é ação exclusiva de `/metas`.

---

## Cor

`#d55181` — a mesma da série Metas no `BUDGET` do [[Tela-Dashboard|Dashboard]],
para que o balde tenha identidade visual consistente.
A barra vira **verde** (`#3ddc84`) **só quando concluída**.

---

## Limites

- **Sem aporte de valor livre.** O aporte é sempre `vl_conta` — não dá para
  "guardar R$ 200 a mais este mês".
- **Sem saldo real.** O progresso é o que foi *marcado*, não o que existe numa
  conta. O sistema não conhece saldo bancário.
- **Sem data-alvo independente das parcelas.** O fim da meta é o fim da
  recorrência.
- **Backend legado sem `pagamentos`** → cai em `conta_paga`: tudo ou nada.

---

## Relacionadas
[[Tipo]] · [[Competencia-e-Pagamento]] · [[Recorrencia]] · [[Tela-Metas]] · [[ADR-003-Metas-derivadas]]
