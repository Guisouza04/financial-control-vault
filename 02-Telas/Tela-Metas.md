---
aliases: ["Tela de Metas"]
tags: [tela]
rota: "/metas"
codigo: ["FinancialControll2.0/src/pages/Metas/index.jsx"]
atualizado: 2026-08-02
---

# Tela · Metas

**Rota:** `/metas` · **Tipo:** 4 · **Componente:** `src/pages/Metas`

Cards de progresso, **não** a tabela do [[ExpenseBox]] — porque uma [[Meta|meta]]
é um aporte acumulativo rumo a um objetivo, não um boleto a vencer.

---

## O que o usuário consegue fazer

| Ação | Detalhe |
|---|---|
| Ver o progresso de cada meta | Barra + `guardado / alvo` + `aportes pagos / total` |
| Registrar aporte | Botão nomeado com **mês e ano** (ex.: "Registrar aporte de Setembro/2026") |
| Criar meta | Modal com recorrência **e ano de término** (`fimAno`) |
| Editar / excluir | Mesmos padrões das outras telas |
| Escolher a competência do aporte | Filtro Ano/Mês no topo |
| Buscar meta pelo nome | Campo na barra de filtros — [[ExpenseBox#Busca — é o ÚLTIMO elo da cadeia de filtros\|mesma busca das tabelas]] |
| Pular para Contas / Investimentos / Opcionais | Barra de abas na linha do título — ver [[FinanceTabs]]. É por onde Metas deixa de ser um destino isolado do [[Nav]] |

---

## As três regras que definem esta tela

### 1. O filtro de mês NÃO filtra a lista

O Ano/Mês do topo só define **de qual competência é o aporte**. A lista mostra
**sempre todas as metas**.

Motivo: uma meta atravessa meses. Se sumisse ao filtrar, o usuário perderia de
vista o próprio objetivo — o oposto do que uma tela de metas deve fazer.

> É a diferença mais importante em relação à [[Tela-Despesas]], onde o filtro
> **é** o recorte da lista.

> **A busca é a única exceção.** O campo de texto realmente encolhe a lista —
> mas é uma lente explícita, que o usuário liga e desliga, não um recorte
> implícito por período. Ela roda **antes** do split em "Em andamento" /
> "Concluídas", para os contadores dos grupos baterem com o que está na tela.

### 2. O botão de aporte segue a meta, não o filtro

`competenciaDoAporte`:
1. Usa o mês do filtro **se** ele fizer parte da janela da meta;
2. Senão, cai em `nextPendingCompetencia()` — o primeiro mês ainda não aportado.

Assim, mover o início de uma meta para setembro faz o botão oferecer
**"Registrar aporte de Setembro/2026"** em vez de travar.

**Com "Todos os meses" o botão continua funcionando** (mira o próximo pendente).
A [[Tela-Despesas]] bloqueia nessa situação — aqui não, porque não há
ambiguidade: a meta tem uma ordem natural de progresso.

O rótulo traz **mês E ano** porque a meta atravessa anos e "Janeiro" sozinho
seria ambíguo.

### 3. Só conta aporte dentro da janela

`goalProgress` filtra os `pagamentos` por `goalCompetencias(account)`.
Sem isso, mover o início de uma meta deixaria aportes órfãos contando como
progresso **para sempre** — bug real, encontrado em uso. Ver [[Meta]].

---

## Progresso é 100% derivado

Não há tabela de metas, nem coluna de alvo ou acumulado:

- **Alvo** = `value × occurrenceCount(account)`
- **Guardado** = `value × aportes na janela`
- **Concluída** = `aportesPagos >= totalAportes`

Ver [[ADR-003-Metas-derivadas]].

---

## Diferenças do modal de meta

| | Despesas | Metas |
|---|---|---|
| Ano de término (`fimAno`) | ❌ fim no ano do início | ✅ pode atravessar o ano |
| [[TagPicker]] | ✅ | ❌ decisão de produto |
| Criável em outras telas | ✅ | ❌ só aqui (`TIPO_OPTIONS` exclui o 4) |

---

## Cor

`#d55181` (mesma da série Metas no `BUDGET`). A barra vira **verde**
(`#3ddc84`) **só quando concluída**.

---

## O que a tela NÃO faz

- **Não aceita aporte de valor livre** — o aporte é sempre `vl_conta`.
- **Não conhece saldo real** — progresso é o que foi marcado, não o que existe
  no banco.
- **Não tem data-alvo separada** das parcelas.
- **Não permite tags.**

---

## Relacionadas
[[Meta]] · [[Competencia-e-Pagamento]] · [[Recorrencia]] · [[Tipo]] · [[Loader]] · [[ADR-003-Metas-derivadas]]
