---
aliases: ["Migrações", "Alembic"]
tags: [backend]
codigo: ["FinancialControllBackend/alembic/versions/"]
atualizado: 2026-08-01
---

# Migrações (Alembic)

O histórico das migrações **é a história do produto** — cada uma marca uma
decisão. Ler nesta ordem dá o contexto de como o sistema chegou aqui.

| # | Nome | O que trouxe | Por quê |
|---|---|---|---|
| `0001` | `init` | `users`, `lancamentos` | Base |
| `0002` | `recorrencia` | `recorrencia`, `dia_vencimento`, `data_inicio`, `data_fim` | Antes disso, repetição era só `qtd_parcelas` a partir de `dt_create` — sem semântica. **Criou o "legado".** Ver [[Recorrencia]] |
| `0003` | `pagamentos` | Tabela `pagamentos` | Um recorrente é 1 linha em N meses → pagamento **não** pode ser campo da linha. Ver [[Competencia-e-Pagamento]] |
| `0004` | `na_fatura` | `lancamentos.na_fatura` | Marcar compra de cartão. Ver [[Fatura-de-Cartao]] |
| `0005` | `data_compra` | `lancamentos.data_compra` | Separar **quando comprou** de **quando pesa no orçamento** |
| `0006` | `fitid` | `lancamentos.fitid` (indexado) | Deduplicar importação de OFX |
| `0007` | `tags` | `tags` + `lancamento_tags` | Categorização transversal ao [[Tipo]]. Ver [[Tag]] |

---

## O padrão: aditivo, nunca destrutivo

Nenhuma migração apagou coluna nem reescreveu dados existentes.

**Consequência deliberada:** registros antigos continuam válidos e são tratados
**em código**, não migrados.

- `recorrencia = NULL` → regra legada de `qtd_parcelas` + `dt_create`
- `conta_paga` continua existindo, mesmo com `pagamentos` sendo a fonte de
  verdade
- `fitid = NULL` em tudo que foi importado antes de `0006`

> **Isso é uma escolha, não descuido.** O custo é a ramificação nos helpers
> (todo `accountActiveInPeriod` trata dois casos); o benefício é nunca ter
> corrompido dado real numa migração de conversão.
>
> **Ao adicionar coluna nova, siga o padrão:** nullable, com fallback em código
> para o valor ausente. Ver [[Guia-Novo-Campo-no-Lancamento]].

---

## Dívida gerada por esse padrão

| Vinda de | Dívida |
|---|---|
| `0002` | Toda lógica de período tem dois caminhos |
| `0003` | `conta_paga` órfão, ainda gravado em alguns fluxos |
| `0006` | Importações pré-`0006` não são dedupláveis — limpeza manual |

Ver [[Pendencias]].

---

## Antes de subir uma versão

🔒 Rode a skill **`/verify-requirements`** antes de qualquer release. Ela varre
o `requirements.txt` procurando incompatibilidades conhecidas entre libs.

Existe por causa de um caso real: `bcrypt 4.x` + `passlib 1.7.4` quebrando o
login **silenciosamente**. Ver [[Autenticacao]].

---

## Relacionadas
[[Modelo-de-Dados]] · [[Recorrencia]] · [[Competencia-e-Pagamento]] · [[Tag]] · [[Fatura-de-Cartao]] · [[Pendencias]]
