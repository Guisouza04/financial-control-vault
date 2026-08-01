---
aliases: ["ADR-001", "naFatura é marcador"]
tags: [adr]
atualizado: 2026-08-01
---

# ADR-001 · `na_fatura` é apenas um marcador

## Contexto

Compras de cartão criam risco de **dupla contagem**. No modelo antigo, o
usuário lançava a fatura inteira como um valor único ("Fatura Nubank —
R$ 3.000"). Se também lançasse compras individuais, tudo somava duas vezes.

A solução da época: `na_fatura = true` fazia o lançamento **não somar** no
total do período. A compra ficava visível, mas fora da conta — a fatura-lump é
que contava.

Aí veio a [[Tela-Importar-Extrato|importação de extrato OFX]], que **itemiza**
as compras. A fatura-lump deixou de existir: agora as compras individuais
*são* a fatura.

Com a regra antiga ainda ativa, **todas as compras importadas sumiam do total**
— o orçamento mostrava saldo livre que não existia.

## Decisão

**`na_fatura` é apenas um MARCADOR visual (💳). O valor conta normalmente no
total do período.**

## Alternativas descartadas

| Alternativa | Por que não |
|---|---|
| Manter "não somar" e não itemizar | Perde o detalhe da compra — a razão de importar extrato |
| Criar entidade "Fatura" que agrupa compras | Modelo bem mais complexo (fechamento, vínculo, reconciliação) para um app pessoal |
| Somar só se não houver fatura-lump no mês | Regra condicional e frágil, difícil de explicar ao usuário |

## Consequências

**Boas**
- O total do mês reflete o gasto real, item a item
- A importação passou a ser confiável
- A regra ficou trivial de explicar: "cartão é um lançamento como outro
  qualquer, só marcado"

**Custos**
- Quem ainda lança a fatura fechada **e** as compras conta duplicado — o
  sistema não protege contra isso
- O checkbox `na_fatura` vale para qualquer [[Tipo|tipo]], o que pode parecer
  sem propósito

**A preservar** ⚠️
> **Não reintroduza lógica de "não somar" em cima de `na_fatura`.** Está
> documentado no próprio código (`sumActiveInPeriod` em `recurrence.js` e no
> model `lancamento.py`) justamente porque a exclusão parece "óbvia" para quem
> chega agora.

---

## Relacionadas
[[Fatura-de-Cartao]] · [[Tela-Importar-Extrato]] · [[Lancamento]]
