---
aliases: ["ADR-007", "Dedup por FITID"]
tags: [adr]
atualizado: 2026-08-01
---

# ADR-007 · Deduplicação de importação por FITID

## Contexto

A [[Tela-Importar-Extrato|importação de extrato]] é **stateless**: não há
tabela de staging, nem registro de "arquivos já importados".

Sem proteção, reimportar o mesmo `.ofx` — ou dois extratos com período
sobreposto — duplicaria todas as compras, e o orçamento do mês inflaria
silenciosamente.

## Decisão

Usar o **FITID** (`Financial Institution Transaction ID`), o identificador único
que o banco atribui a cada transação no padrão OFX. Guardado em
`lancamentos.fitid` (indexado, migração `0006`).

- **Preview:** FITID já existente → `ja_importada = true`; a linha vem
  **desmarcada**, com badge
- **Commit:** pula FITIDs já gravados **ou repetidos no mesmo lote**; devolve
  `{ created, skipped }`

## Alternativas descartadas

| Alternativa | Por que não |
|---|---|
| Hash de `(data, valor, descrição)` | Duas compras iguais no mesmo dia no mesmo lugar são legítimas — seriam tratadas como duplicata |
| Tabela de arquivos importados | Não protege contra extratos com período sobreposto (arquivos diferentes, transações repetidas) |
| Confiar na revisão manual | O usuário não tem como saber o que já importou meses atrás |
| Constraint `UNIQUE` no banco | Erro no meio do lote em vez de "pular e seguir"; e FITID nulo é válido |

## Consequências

**Boas**
- Reimportar o mesmo arquivo é **seguro**
- Extratos sobrepostos funcionam
- O usuário vê o que foi pulado e por quê

**Custos / furos conhecidos** ⚠️

1. **Transação sem FITID não é deduplicada.** Nem todo OFX traz o campo
   (`fitid` é `str | None`). Reimportar duplica esses lançamentos.
2. **Registros anteriores à migração `0006`** têm `fitid = NULL` → não são
   detectados. Limpeza é manual.
3. **FITID não é único entre bancos.** Hoje não dá problema porque não há
   entidade "conta bancária" — mas se um dia houver múltiplas contas, a
   unicidade correta seria `(banco, fitid)`.

**A preservar** ⚠️
> - **Não crie `UNIQUE` em `fitid` sozinho.** Nulos são válidos e esperados, e
>   o comportamento desejado é *pular*, não *falhar*.
> - **Se adicionar suporte a múltiplas contas/bancos**, a chave de dedup precisa
>   virar composta.

---

## Relacionadas
[[Fatura-de-Cartao]] · [[Tela-Importar-Extrato]] · [[Migracoes]]
