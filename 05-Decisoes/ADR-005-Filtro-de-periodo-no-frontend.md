---
aliases: ["ADR-005", "Filtro no frontend"]
tags: [adr]
atualizado: 2026-08-01
---

# ADR-005 · O filtro por período é client-side

## Contexto

Originalmente `GET /financas/{tipo}` filtrava por `dt_create` — devolvia só os
lançamentos **criados** no mês consultado.

Isso quebrou assim que [[Recorrencia|recorrência]] entrou: uma conta mensal
criada em janeiro **desaparecia** de fevereiro em diante, porque `dt_create`
continuava sendo janeiro. O lançamento existia, estava ativo, e não aparecia.

Filtrar corretamente no backend exigiria reimplementar em SQL toda a lógica de
`UNICA`/`MENSAL`/`ANUAL` **mais** a regra legada — e mantê-la em sincronia com
a versão JavaScript, que a UI precisa de qualquer forma (para rótulo de parcela,
competência, progresso de meta).

## Decisão

**O backend devolve todos os lançamentos do usuário para o tipo.
O frontend decide o que aparece em cada período**, via
`accountActiveInPeriod` (`src/utils/recurrence.js`).

## Alternativas descartadas

| Alternativa | Por que não |
|---|---|
| Reimplementar a regra em SQL | Duas implementações da regra mais complexa do sistema, garantidas a divergir |
| Materializar parcelas no banco | Ver [[ADR-002-Pagamento-por-competencia]] — mesmo motivo |
| Endpoint que expande ocorrências server-side | Move a complexidade sem eliminá-la; o front ainda precisaria da lógica |

## Consequências

**Boas**
- **Uma única implementação** da regra de período, em JS, testável e visível
- [[Tela-Dashboard|Dashboard]] e telas de despesa somam **exatamente** o mesmo
  conjunto — ambos passam por `accountActiveInPeriod`
- Trocar de mês no filtro é instantâneo: sem round-trip

**Custos**
- **Não escala.** Todo lançamento do usuário trafega a cada carga. Para um app
  pessoal (centenas de registros) é irrelevante; para milhares, não seria
- O Dashboard faz **4 requisições** (uma por tipo) no mount
- Sem paginação server-side

**A preservar** ⚠️
> - **Use `accountActiveInPeriod`.** Reimplementar o filtro numa tela nova é o
>   caminho garantido para o Dashboard discordar da listagem — e a divergência
>   é silenciosa.
> - **Não volte a filtrar por `dt_create` no backend.** Foi exatamente o bug
>   original.

**Quando revisitar:** se o volume por usuário passar de alguns milhares de
lançamentos, ou se surgir multiusuário/família.

---

## Relacionadas
[[Recorrencia]] · [[Endpoints]] · [[Tela-Dashboard]] · [[ExpenseBox]] · [[Camada-de-Dados]]
