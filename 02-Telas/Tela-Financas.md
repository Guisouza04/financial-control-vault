---
aliases: ["Finanças", "Hub de Finanças"]
tags: [tela]
rota: "/Financas"
codigo: ["FinancialControll2.0/src/pages/Despesas/index.jsx"]
atualizado: 2026-08-01
---

# Tela · Finanças (hub)

**Rota:** `/Financas` · **Componente:** `src/pages/Despesas` (nome legado)

Tela puramente de navegação. Quatro cards, nenhuma lógica, nenhuma chamada de
API.

| Card | Leva para |
|---|---|
| Contas | [[Tela-Despesas\|/Contas]] |
| Investimentos | [[Tela-Despesas\|/Investments]] |
| Opcionais | [[Tela-Despesas\|/Optional]] |
| Importar Extrato | [[Tela-Importar-Extrato\|/importar]] |

> **Metas não está aqui.** `/metas` é item de primeiro nível no [[Nav]], porque
> é um dos quatro baldes e não uma subseção de despesas.

---

## Detalhe de UI que tem razão de ser

Os três primeiros cards usam `variant="preencherFill"`; o de **Importar Extrato
não**.

Motivo: o ícone de importar é de **contorno** (`fill="none"`). Preencher os
paths no hover transformaria o ícone num borrão escuro. Sem a variante, só o
traço muda de cor.

> Se você trocar esse ícone por um sólido, aí sim adicione `preencherFill`.

---

## Nome do componente ≠ nome da tela

O arquivo é `pages/Despesas`, a rota é `/Financas` e o título é "Finanças".
Resquício de quando a tela era "Despesas". **Não é bug** — só confusão em
potencial ao procurar o arquivo.

---

## Relacionadas
[[Tela-Despesas]] · [[Tela-Importar-Extrato]] · [[Nav]] · [[Tipo]]
