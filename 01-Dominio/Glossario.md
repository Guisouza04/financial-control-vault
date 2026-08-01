---
aliases: ["Glossário", "Vocabulário"]
tags: [dominio]
atualizado: 2026-08-01
---

# Glossário

O vocabulário do Financial Control. Várias dessas palavras têm significado
**mais estreito** aqui do que no português comum — é justamente onde nascem os
mal-entendidos.

| Termo | No sistema significa | Cuidado |
|---|---|---|
| **[[Lancamento\|Lançamento]]** | Qualquer registro financeiro. Tabela `lancamentos`. | Uma **meta também é um lançamento** (tipo 4). Não existe tabela de metas. |
| **[[Tipo]]** | O balde do orçamento: 1=Contas, 2=Investimentos, 3=Opcionais, 4=Metas | É o *plano* (60/20/10/10), não a natureza do gasto. Essa é a [[Tag]]. |
| **[[Tag]]** | Rótulo livre do usuário ("Mercado", "Combustível") | **Transversal ao tipo.** Um lançamento pode ter várias. |
| **[[Recorrencia\|Recorrência]]** | Como o lançamento se repete: `UNICA`, `MENSAL`, `ANUAL` | `NULL` = registro **legado**, tratado por regra antiga. |
| **[[Competencia-e-Pagamento\|Competência]]** | `"YYYY-MM"` — identifica **uma parcela** de um recorrente | É a chave do pagamento. Sem ela não dá pra saber *qual* mês foi pago. |
| **Parcela / Ocorrência** | Uma repetição do lançamento em um mês | 1 linha no banco ↔ N parcelas na tela. |
| **Aporte** | Numa [[Meta]], "parcela paga" lê-se **aporte feito** | Mesmo mecanismo, nome diferente por contexto. |
| **Pendente** | Parcela **sem** linha em `pagamentos` para aquela competência | Pendente é *ausência* de registro, não um registro com `pago=false`. |
| **[[Salario\|Salário]]** | Renda mensal do usuário (`users.salario`) | Base de **todo** o Dashboard. Sem ele, os medidores ficam sem limite. |
| **Comprometido** | Soma de tudo que está ativo no mês selecionado | Inclui compras de cartão (`na_fatura`). Ver [[Fatura-de-Cartao]]. |
| **Saldo livre** | `salário − comprometido` | Pode ser negativo. |
| **`na_fatura`** | Marcador 💳 de compra de cartão | **Não** exclui do total. Ver [[ADR-001-naFatura-e-marcador]]. |
| **`data_compra`** | Quando a compra foi feita (ex.: 30/06) | ≠ período do lançamento, que é o mês do **vencimento da fatura**. |
| **FITID** | ID único da transação no arquivo OFX | Usado para **deduplicar** importação. |
| **Legado** | Lançamento sem `recorrencia` preenchida | Criado antes da migração `0002`. Segue a regra `qtd_parcelas` + `dt_create`. |
| **Bucket** | Sinônimo de [[Tipo]] no código do Dashboard | Config `BUDGET` em `src/pages/Home`. |
| **Teto vs. Meta** | Tipo de limite do bucket | `teto` = não pode passar (Contas, Opcionais). `meta` = alvo a atingir (Investimentos, Metas). |

---

## Distinções que mais confundem

### Tipo ≠ Tag
O **tipo** responde *"de qual fatia do salário isso sai?"* — é orçamento.
A **tag** responde *"o que é isso?"* — é natureza.

Mercado pode ser tipo 1 (Contas, o mercado do mês) ou tipo 3 (Opcionais, aquele
supérfluo), e nos dois casos leva a tag `Mercado`. Por isso o Dashboard tem
duas visões distintas: distribuição por bucket **e** gastos por tag.

### Lançamento ≠ Parcela
Uma conta mensal de 12 meses é **uma** linha em `lancamentos` e **doze**
parcelas na experiência. Todo bug de pagamento que já apareceu nasceu de tratar
esses dois como a mesma coisa. Ver [[Competencia-e-Pagamento]].

### Pago ≠ `conta_paga`
`conta_paga` é o campo **legado**, único por lançamento — tudo ou nada. O status
real vive na tabela `pagamentos`, por competência. O campo antigo só é
consultado quando `pagamentos` não vem do backend.
