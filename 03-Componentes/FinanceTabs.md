---
aliases: ["FinanceTabs", "Abas de finanças", "Barra de seções"]
tags: [componente]
codigo: ["FinancialControll2.0/src/components/FinanceTabs/index.jsx", "FinancialControll2.0/src/utils/financeTypes.js"]
atualizado: 2026-08-02
---

# FinanceTabs

**`src/components/FinanceTabs`**

Barra de abas das **quatro seções de finança** — Contas · Investimentos ·
Opcionais · Metas — na linha do título de [[Tela-Despesas]] e [[Tela-Metas]].

---

## O problema que ela resolve

Os quatro baldes do plano 60/20/10/10 são **irmãos**, e o uso é comparativo:
olhar Contas e, na sequência, querer ver quanto sobrou em Opcionais. A navegação
tratava cada um como uma ilha:

| De → para | Antes | Agora |
|---|---|---|
| Contas → Investimentos | "Voltar" → hub → card | 1 clique |
| Contas → Metas | [[Nav]] → Metas (Metas nem está no hub) | 1 clique |

A barra também **mostra que as quatro telas existem e são do mesmo grupo** —
coisa que o hub, exibindo três cards e escondendo Metas, não faz.

---

## Sem props: a aba ativa vem da URL

O componente não recebe qual seção está aberta. Se recebesse, uma tela poderia
se declarar a seção errada e a barra mentiria. `NavLink` resolve pelo
`location.pathname`.

> **É o primeiro `NavLink` do projeto.** O [[Nav]] lateral usa `Link` e **não**
> destaca o item ativo. Se um dia o Nav ganhar destaque de item atual, é o mesmo
> caminho.

---

## ⚠️ Não passe `caseSensitive`

`NavLink` tem a prop `caseSensitive` com default **`false`** — ele compara os dois
lados em minúsculo. É **só por isso** que o destaque funciona com as rotas de
caixa mista do projeto: `/Contas` capitalizada e `/metas` minúscula
(ver [[ADR-006-Caixa-das-rotas]]).

Ligar a flag apagaria o destaque em parte das telas — sem erro, sem 404, só a
aba não acendendo. Difícil de diagnosticar depois.

---

## Importar Extrato fica de fora

É uma **ação**, não um tipo. Continua sendo o quarto card do hub
([[Tela-Financas]]), que segue como porta de entrada e único acesso a
`/importar`. O botão "Voltar" das telas também continua no lugar.

---

## `financeTypes.js` — a fonte única de tipo → tela

Rótulos, rotas e cores saíram do `BUDGET` do [[Tela-Dashboard|Dashboard]] para
`src/utils/financeTypes.js`. Sem isso a barra seria a **quinta** cópia do mapa
"tipo → rótulo" no front.

```js
FINANCE_TYPES = [{ tipo, label, labelSingular, route, color }, ...]
financeTypeOf(tipo)
```

O `BUDGET` faz spread daqui e mantém local só `pct` e `kind` — que são regra de
**orçamento**, não identidade do tipo. Assim a rota do 👁 e a rota da aba não
podem divergir.

> **Ainda duplicado:** os `TIPO_OPTIONS` dos modais ([[ExpenseBox]],
> [[QuickAddModal]], [[Tela-Importar-Extrato]]) são listas próprias, no formato
> `{ value: "1", label: "Conta" }`. Dá para derivá-los de `labelSingular` —
> limpeza pendente, não bug.

---

## Detalhes visuais com razão de ser

- **O ponto colorido é reforço, não a informação.** O rótulo carrega o
  significado; em daltonismo ou monocromático nada se perde. Mesma paleta
  categórica dos medidores do Dashboard.
- **`.pageHead`** (classe global, junto de `.containerExpenses`) põe título e
  abas na mesma linha. Como bloco separado, o `gap: 4rem` do container as
  afastaria do título e empurraria a tabela para baixo.
- **No estreito as abas quebram de linha**, não rolam na horizontal — scroll
  esconderia destino, que é justamente o que a barra existe para revelar.

---

## Limite conhecido

**O período não acompanha a troca de aba.** Sair de Contas/Agosto e cair em
Investimentos/mês atual mantém parte do atrito. A saída é levar `ano`/`mes` na
URL e o [[ExpenseBox]] lê de lá — trabalho em aberto, ver [[Pendencias]].

---

## Relacionadas
[[Tela-Despesas]] · [[Tela-Metas]] · [[Tela-Financas]] · [[Nav]] · [[Tipo]] ·
[[ADR-006-Caixa-das-rotas]]
