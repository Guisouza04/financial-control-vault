---
aliases: ["Dashboard", "Home", "Tela inicial"]
tags: [tela]
rota: "/"
codigo: ["FinancialControll2.0/src/pages/Home/index.jsx"]
atualizado: 2026-08-01
---

# Tela · Dashboard

**Rota:** `/` · **Componente:** `src/pages/Home`

A tela inicial e o **produto em si**: todo o resto do sistema existe para
alimentar esta visão. Ela responde uma pergunta só —
**"meu mês está aderente ao plano 60/20/10/10?"**

---

## O que ela entrega

| Seção | Responde |
|---|---|
| **KPIs** | Salário, comprometido, saldo livre, % em Contas |
| **Medidores** | Cada bucket está dentro do limite? (gasto × limite) |
| **Donut + legenda** | Como o dinheiro se distribui **de fato** × como deveria |
| **Gastos por tag** | *Em que* o dinheiro foi, independente de bucket |

Layout: `InsightGrid` de duas colunas (medidores + distribuição lado a lado),
colapsando para uma coluna abaixo de **1024px**.

---

## O que o usuário consegue fazer

- **Filtrar mês/ano** (default: atual), com navegação de mês
- **Criar lançamento sem sair da tela** — botão ⚡ (`QuickAddButton`) abre o
  [[QuickAddModal]]. Ao salvar, `onCreated(tipo)` refaz o fetch **daquele tipo**,
  atualizando o resumo na hora.
- Navegar para as telas de detalhe pelos links dos estados vazios

---

## De onde vêm os dados

```
useAccounts(1) ─┐
useAccounts(2) ─┼─→ sumActiveInPeriod(accounts, ano, mes) → medidores/donut
useAccounts(3) ─┤   accountActiveInPeriod + account.tags  → gastos por tag
useAccounts(4) ─┘
financeService.fetchSalary() ────────────────────────────→ KPIs e limites
```

> **Os quatro hooks buscam SEM filtro**, uma vez cada. A soma por período é
> client-side. É o mesmo caminho que as telas de despesa usam —
> ver [[ADR-005-Filtro-de-periodo-no-frontend]].
>
> **Consequência:** se o Dashboard e a listagem discordarem, alguém
> reimplementou o filtro em vez de usar `accountActiveInPeriod`.

---

## `BUDGET` — a configuração central

```js
{ tipo: 1, label: "Contas",        pct: 60, kind: "teto", color: "#3987e5" }
{ tipo: 2, label: "Investimentos", pct: 20, kind: "meta", color: "#199e70" }
{ tipo: 3, label: "Opcionais",     pct: 10, kind: "teto", color: "#c98500" }
{ tipo: 4, label: "Metas",         pct: 10, kind: "meta", color: "#d55181" }
```

**`kind` é a regra que mais importa:** `teto` fica vermelho ao **exceder**;
`meta` fica verde ao **atingir**. A mesma barra em 80% lê-se de formas opostas.
Ver [[Tipo]].

**Cores de status** (`STATUS`: `good #3ddc84`, `warning #fab219`,
`critical #ff6b6b`) **nunca** são reutilizadas como cor de série — senão um
bucket ficaria verde por acaso e leria como aprovação.

---

## Gastos por tag — a seção com a pegadinha

Ranking horizontal do comprometido por [[Tag|tag]] no mês, somando **todos os
tipos**.

Como um lançamento pode ter **várias** tags, **o valor conta em cada uma** —
então **a soma do ranking pode passar do total comprometido**. Isso é esperado
e está avisado no subtítulo. Não "conserte" dividindo o valor entre as tags:
isso responderia uma pergunta diferente (e menos útil).

- Sem tag → bucket neutro **"Sem tag"** (`#7a7a85`, nunca colide com tag real)
- Barras escaladas pela maior fatia
- Cores vêm da própria tag
- Sem lançamento tagueado no mês → estado vazio com link para Finanças

---

## Sem salário configurado

Estado de primeira classe, não erro. Ver [[Salario]].

- Aviso (`InlineNotice`) com link para `/dados`
- **Medidores ficam sem limite** — não há denominador
- **Donut continua funcionando** — proporção entre buckets não depende do salário

---

## O que a tela NÃO faz

- **Não compara meses.** Não há série temporal, tendência ou "vs. mês anterior".
- **Não projeta.** Nada de previsão de saldo futuro.
- **Não permite editar** lançamento — só criar (⚡). Editar é nas telas de detalhe.
- **Não usa o salário histórico.** Usa o salário **atual** para qualquer mês
  consultado. Ver [[Salario]].
- **Não filtra por tag.** A seção de tags é leitura, não filtro.

---

## Relacionadas
[[Tipo]] · [[Salario]] · [[Tag]] · [[QuickAddModal]] · [[Camada-de-Dados]] · [[ADR-005-Filtro-de-periodo-no-frontend]]
