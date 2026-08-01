---
aliases: ["ADR-003", "Metas derivadas"]
tags: [adr]
atualizado: 2026-08-01
---

# ADR-003 · Metas sem tabela própria, progresso derivado

## Contexto

[[Meta|Metas]] são o 4º balde do orçamento. O reflexo natural seria criar uma
tabela `metas` com `valor_alvo` e `valor_acumulado`.

Mas ao olhar o modelo existente, **tudo já estava lá**: uma meta é um aporte
recorrente (valor × ocorrências = alvo) e "aporte feito" é exatamente o que a
tabela `pagamentos` já registra.

## Decisão

**Nenhuma migração.** Meta é um [[Lancamento|lançamento]] com `tipo = 4`.
Alvo e progresso são **derivados em tempo de leitura**:

```
alvo     = value × occurrenceCount(account)
guardado = value × aportes dentro de goalCompetencias(account)
```

"Parcela paga" lê-se **"aporte feito"**.

## Alternativas descartadas

| Alternativa | Por que não |
|---|---|
| Tabela `metas` com alvo e acumulado | Dado redundante que **pode divergir** da verdade (os aportes). Exige manter sincronizado a cada operação |
| Coluna `valor_acumulado` em `lancamentos` | Mesmo problema, e polui a tabela para os outros 3 tipos |
| Meta como tipo especial com endpoints próprios | Duplicaria CRUD, filtro e recorrência inteiros |

## Consequências

**Boas**
- **Impossível divergir**: não há acumulado guardado que possa ficar errado
- Metas herdaram de graça recorrência, edição, exclusão e o toggle de pagamento
- Zero migração, zero backfill

**Custos**
- O progresso é **recalculado a cada render** (irrelevante nesta escala)
- Metas ficaram acopladas às regras de [[Recorrencia]] — mudar recorrência mexe
  em metas sem parecer que mexe
- Precisou de `fimAno` opcional para a meta atravessar o ano sem afetar as
  telas de despesa

**O bug que essa decisão expôs** ⚠️

Derivar do `pagamentos` significa que **aportes fora da janela atual também
apareciam**. Caso real:

> Meta começava em julho, recebeu aporte (`pagamentos: ["2026-07"]`). O usuário
> moveu o início para setembro. O aporte de julho ficou **órfão** — fora da nova
> janela — mas seguia contando como progresso **para sempre**.

Correção: `goalCompetencias(account)` lista a janela, e `goalProgress` conta
**só** os pagamentos que caem nela.

**A preservar** ⚠️
> - **`goalProgress` deve continuar filtrando pela janela.** Remover o filtro
>   reintroduz o bug — e ele é silencioso: o progresso só fica errado, nada
>   quebra.
> - **Não crie tabela de metas** para "simplificar". A simplicidade aqui está
>   justamente em não ter estado duplicado.

---

## Relacionadas
[[Meta]] · [[Tela-Metas]] · [[Competencia-e-Pagamento]] · [[ADR-002-Pagamento-por-competencia]]
