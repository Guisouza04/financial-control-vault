---
aliases: ["Fatura de Cartão", "Cartão de crédito", "naFatura"]
tags: [dominio]
codigo: ["FinancialControll2.0/src/pages/ImportarExtrato/index.jsx", "FinancialControllBackend/app/services/ofx.py"]
atualizado: 2026-08-01
---

# Fatura de Cartão

Compras de cartão são o caso em que **quando você gastou** e **quando isso pesa
no orçamento** são meses diferentes. O sistema resolve isso com duas datas.

---

## As duas datas

| | Campo | Exemplo | Papel |
|---|---|---|---|
| **Quando comprou** | `data_compra` | 30/06 | Só informação. Não afeta período. |
| **Quando conta no orçamento** | `data_inicio` / `data_fim` | agosto | O **mês do vencimento da fatura** |

Uma compra feita em 30/06, numa fatura que vence em agosto, é um
[[Lancamento|lançamento]] **de agosto** — porque é em agosto que ela consome
o orçamento. `data_compra = 2026-06-30` fica guardada como referência.

> **Confundir as duas é o erro clássico.** Se alguém filtrar/somar por
> `data_compra`, o Dashboard passa a mostrar gasto no mês errado.

Campo adicionado na migração `0005_data_compra`.

---

## `na_fatura` é apenas um MARCADOR

`na_fatura = true` significa "isto é uma compra de cartão" e renderiza o
ícone 💳. **Só isso.**

**O valor conta normalmente no total do período.**

Nem sempre foi assim. A regra antiga era "não somar", para evitar dupla
contagem quando a fatura inteira era lançada como um valor único: se você
lançasse "Fatura Nubank R$ 3.000" *e* as compras individuais, tudo somaria duas
vezes.

Quando passamos a **itemizar** as compras via [[Tela-Importar-Extrato|importação
de extrato]], a fatura-lump deixou de existir — e a regra de exclusão virou
justamente a causa do erro (as compras itemizadas sumiam do total). Foi
aposentada. Ver [[ADR-001-naFatura-e-marcador]].

> **Não reintroduza lógica de "não somar" em cima de `na_fatura`.** O checkbox
> vale para qualquer [[Tipo|tipo]] e é puramente visual.

---

## FITID e deduplicação

Cada transação num arquivo OFX tem um `FITID` — identificador único atribuído
pelo banco. O sistema guarda esse valor no lançamento importado
(migração `0006_fitid`).

**No preview:** transação cujo FITID já existe vem com `ja_importada = true`; a
UI mostra o badge "já importada" e **deixa a linha desmarcada**.

**No commit:** FITIDs já gravados — ou repetidos dentro do mesmo lote — são
**pulados**. A resposta traz `{ created, skipped }`.

### Dois furos conhecidos

1. **Transação sem FITID não é deduplicada.** Alguns OFX não trazem o campo.
   Reimportar o mesmo arquivo duplica esses lançamentos.
2. **Registros importados antes da migração `0006`** não têm `fitid` → não são
   detectados como duplicata. Limpeza é manual.

---

## O que não existe

- **Não há entidade "fatura".** Não existe tabela, nem agrupamento, nem
  "fechar fatura". Só lançamentos marcados.
- **Não há cartão cadastrado.** O sistema não sabe quais cartões você tem, nem
  limite, nem bandeira.
- **Não há cálculo automático de vencimento.** O usuário informa a data de
  vencimento manualmente a cada importação.
- **Não há parcelamento de compra de cartão.** Cada transação importada vira um
  lançamento `UNICA`. Uma compra em 10x aparece como 10 transações separadas nas
  respectivas faturas (se estiverem no extrato) — o sistema não as relaciona.

---

## Relacionadas
[[Lancamento]] · [[Tela-Importar-Extrato]] · [[ADR-001-naFatura-e-marcador]] · [[Migracoes]]
