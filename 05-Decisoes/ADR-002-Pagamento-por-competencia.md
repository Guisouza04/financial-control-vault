---
aliases: ["ADR-002", "Pagamento por competência"]
tags: [adr]
atualizado: 2026-08-01
---

# ADR-002 · Pagamento por competência, em tabela própria

## Contexto

Um [[Lancamento|lançamento]] recorrente é **uma linha** no banco que aparece em
**vários meses**. O status de pagamento era `lancamentos.conta_paga` — um campo
`"S"`/`"N"` na linha.

Isso é logicamente impossível de sustentar: marcar a conta de luz de agosto
como paga marcava **todos** os meses, porque só existe um campo.

## Decisão

**Modelo (A):** tabela `pagamentos(lancamento_id, competencia)`, onde
`competencia` é `"YYYY-MM"`. Marcar faz *insert*; desmarcar faz *delete*.
**Ausência de linha = pendente.**

## Alternativas descartadas

| Alternativa | Por que não |
|---|---|
| **(B)** Materializar cada parcela como linha em `lancamentos` | Explode o volume; editar o recorrente vira migração de N linhas; quebra a ideia de "um lançamento" |
| **(C)** Campo JSON com lista de competências pagas | Sem constraint de unicidade, sem índice, difícil de consultar |
| **(D)** Linha com `pago: boolean` | Permite estado inconsistente (duas linhas discordando) e cria linha para o caso mais comum (pendente) |

## Consequências

**Boas**
- Cada parcela é quitada de forma independente, que é a realidade do problema
- Tabela pequena: só o que foi pago existe
- `UniqueConstraint(lancamento_id, competencia)` torna duplicata impossível

**Custos**
- Surgiu o conceito de **competência ambígua**: um `MENSAL` sem mês no filtro
  não identifica parcela → `occurrenceCompetencia` devolve `null`, e **toda UI
  de pagamento precisa tratar isso**
- **Pagamento órfão:** editar o período não limpa `pagamentos`. Inofensivo em
  despesas, mas causou bug real em [[Meta|metas]] — resolvido por
  `goalCompetencias`. Ver [[ADR-003-Metas-derivadas]]
- `conta_paga` continua no modelo como fallback legado

**A preservar** ⚠️
> - **Pendente é ausência de linha.** Não crie registro com `pago = false`.
> - **Toda nova UI que marque pagamento precisa tratar `competencia === null`.**
>   [[ExpenseBox]] bloqueia; [[Tela-Metas]] mira o próximo pendente. As duas
>   respostas são válidas — a que não é válida é ignorar o `null`.
> - **Não escreva lógica nova sobre `conta_paga`.**

---

## Relacionadas
[[Competencia-e-Pagamento]] · [[Recorrencia]] · [[ExpenseBox]] · [[Migracoes]]
