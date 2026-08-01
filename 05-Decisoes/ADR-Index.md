---
aliases: ["Índice de ADRs", "Decisões"]
tags: [adr]
atualizado: 2026-08-01
---

# Índice de ADRs

**ADR = Architecture Decision Record.** Uma decisão com trade-off, registrada
com o contexto que a motivou, as alternativas descartadas e o que ela custa.

> **Para que servem aqui:** impedir que uma decisão pensada seja desfeita por
> alguém (eu incluído) que só viu o código e achou estranho. Se algo parece
> errado mas tem ADR, leia antes de "consertar".

---

| # | Decisão | Status |
|---|---|---|
| [[ADR-001-naFatura-e-marcador\|001]] | `naFatura` é só marcador, não exclui do total | ✅ Vigente |
| [[ADR-002-Pagamento-por-competencia\|002]] | Pagamento por competência, em tabela própria | ✅ Vigente |
| [[ADR-003-Metas-derivadas\|003]] | Metas sem tabela própria — progresso derivado | ✅ Vigente |
| [[ADR-004-Controles-customizados\|004]] | Componentes próprios no lugar de `<select>`/`<input date>` | ✅ Vigente |
| [[ADR-005-Filtro-de-periodo-no-frontend\|005]] | Filtro por período é client-side | ✅ Vigente |
| [[ADR-006-Caixa-das-rotas\|006]] | Caixa das rotas é inconsistente — e tudo bem | ✅ Vigente |
| [[ADR-007-Dedup-por-FITID\|007]] | Dedup de importação por FITID | ✅ Vigente |

---

## Quando escrever um ADR novo

Escreva quando **daqui a seis meses alguém puder perguntar "por que não fizeram
do jeito óbvio?"**. Sinais:

- Você descartou uma abordagem mais simples por um motivo não visível no código
- A decisão gera trabalho recorrente (ramificação, compatibilidade, duplicação)
- Houve um bug real que motivou a mudança
- Duas telas tratam a mesma situação de formas diferentes **de propósito**

## Template

```markdown
# ADR-NNN · Título

## Contexto
O que existia e qual pressão apareceu.

## Decisão
O que foi decidido, em uma frase.

## Alternativas descartadas
| Alternativa | Por que não |

## Consequências
**Boas:** …
**Custos:** …
**A preservar:** o que não pode ser desfeito por engano.
```
