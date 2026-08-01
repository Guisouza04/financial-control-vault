---
aliases: ["Lançamento", "Lancamentos"]
tags: [dominio]
codigo: ["FinancialControllBackend/app/models/lancamento.py", "FinancialControll2.0/src/services/financeService.js"]
atualizado: 2026-08-01
---

# Lançamento

A **unidade atômica do sistema**. Tudo é lançamento: uma conta de luz, um aporte
em investimento, um café, uma meta de viagem. Uma única tabela (`lancamentos`)
guarda todos, diferenciados pelo [[Tipo]].

> **Decisão estruturante:** não há tabela separada para metas, para despesas ou
> para compras de cartão. Um modelo, discriminado por `tipo` e por flags.
> Isso é o que permite o Dashboard somar tudo com a mesma função.

---

## Campos e o que cada um carrega

| Campo | Tipo | Significa |
|---|---|---|
| `id` | int | PK |
| `de_conta` | str(255) | Nome/descrição. É o que aparece na tabela. |
| `vl_conta` | Numeric(10,2) | Valor **de uma ocorrência**, não o total. |
| `tipo` | int | 1–4. Ver [[Tipo]]. |
| `qtd_parcelas` | int | Total de ocorrências. |
| `recorrencia` | str? | `UNICA` / `MENSAL` / `ANUAL`. **NULL = legado.** |
| `dia_vencimento` | int? | 1–31 |
| `data_inicio` | date? | Primeira ocorrência |
| `data_fim` | date? | Última data considerada |
| `conta_paga` | str(1) | `"S"`/`"N"` — **legado**, ver [[Competencia-e-Pagamento]] |
| `na_fatura` | bool | Marcador 💳 de compra de cartão |
| `data_compra` | date? | Data original da compra no cartão |
| `fitid` | str(64)? | ID da transação OFX, para dedup |
| `dt_create` | datetime | Criação |
| `user_id` | FK | Dono. `ondelete=CASCADE`. |

**Relações:**
- `pagamentos` → N linhas, uma por competência quitada ([[Competencia-e-Pagamento]])
- `tags` → N:N via `lancamento_tags` ([[Tag]])

---

## `vl_conta` é o valor da ocorrência

Uma conta de R$ 500 por 12 meses guarda `vl_conta = 500`, não 6000. O total é
sempre derivado (`valor × ocorrências`). Isso vale para [[Meta|metas]] também —
é o que torna o alvo da meta calculável sem campo próprio.

> **Na API o valor trafega como string** com ponto decimal (`"1234.56"`). A
> máscara BRL do input é só exibição (`src/utils/currency.js`).

---

## Os dois "sabores" de lançamento

### Com recorrência (atual)
Tem `recorrencia`, `data_inicio`, `data_fim`, `dia_vencimento`. A ocorrência em
cada mês é calculada por [[Recorrencia]].

### Legado (sem recorrência)
Criado antes da migração `0002_recorrencia`. `recorrencia` é `NULL` e o sistema
cai numa regra antiga: janela de meses consecutivos a partir de `dt_create`
(no front, `creationMonth`) com duração `qtd_parcelas`.

**Todo código que interpreta período precisa tratar os dois.** Os helpers de
`src/utils/recurrence.js` já encapsulam isso — use `accountActiveInPeriod` em
vez de reimplementar a regra. Ver [[Guia-Antes-de-Implementar]].

---

## Shape na fronteira front/back

O backend fala `snake_case` em português; o frontend trabalha com um shape
normalizado. A tradução vive em **um lugar só**: `transformAccount()` em
`src/services/financeService.js`.

```
API                     →  Frontend
de_conta                →  name
vl_conta (string)       →  value (number)
qtd_parcelas            →  durationMonths
conta_paga              →  contaPaga
dt_create               →  creationMonth ("YYYY-MM")
data_inicio/data_fim    →  dataInicio/dataFim
dia_vencimento          →  diaVencimento
data_compra             →  dataCompra
na_fatura               →  naFatura
pagamentos              →  pagamentos (["YYYY-MM"])
tags                    →  tags ([{id, nome, cor}])
```

> **Se você adicionar um campo, ele precisa passar por aqui** — senão chega no
> backend mas some no caminho de volta. Ver [[Guia-Novo-Campo-no-Lancamento]].

---

## O que um lançamento NÃO tem

- **Não tem categoria fixa** além do tipo — categorização é via [[Tag]].
- **Não tem moeda.** Tudo é BRL, implicitamente.
- **Não tem anexo, comprovante ou observação.** Não há campo de texto livre
  além do nome.
- **Não tem histórico de alteração.** Editar sobrescreve.
- **Não tem valor variável por parcela.** Todas as ocorrências valem o mesmo.
  Uma conta de luz que varia todo mês não é representável — a alternativa é
  criar lançamentos `UNICA` por mês.

Ver [[O-que-o-sistema-nao-faz]].

---

## Relacionadas
[[Tipo]] · [[Recorrencia]] · [[Competencia-e-Pagamento]] · [[Tag]] · [[Meta]] · [[Modelo-de-Dados]]
