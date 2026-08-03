---
aliases: ["Importar Extrato", "Importação OFX"]
tags: [tela]
rota: "/importar"
codigo: ["FinancialControll2.0/src/pages/ImportarExtrato/index.jsx", "FinancialControllBackend/app/services/ofx.py"]
atualizado: 2026-08-01
---

# Tela · Importar Extrato

**Rota:** `/importar` (card no hub [[Tela-Financas|Finanças]])

Importa um extrato **OFX de cartão de crédito** e transforma cada transação em
um [[Lancamento|lançamento]]. O fluxo é **stateless com revisão manual** — nada
é gravado sem o usuário confirmar.

---

## O fluxo, em 4 passos

```mermaid
graph LR
    A["1 · Selecionar .ofx"] --> B["2 · Preview<br/>(não grava)"]
    B --> C["3 · Revisar<br/>marcar/desmarcar + tipo"]
    C --> D["4 · Informar vencimento<br/>e confirmar"]
    D --> E[("Lançamentos ÚNICA")]
```

### 1 · Leitura do arquivo
`readOfxText(file)` lê respeitando o **charset do cabeçalho**:
OFX 1.x (SGML) costuma ser Windows-1252/Latin-1; 2.x é UTF-8.

> **Ler tudo como UTF-8 corrompe acentos.** Descrições de estabelecimento
> brasileiro viram lixo. Esse tratamento é intencional — não simplifique.

### 2 · Preview
`importPreview(content)` → `POST /financas/import/preview`.
O backend faz o parse (`app/services/ofx.py`, **sem lib externa**) e devolve as
transações **sem gravar nada**.

> O conteúdo vai como **texto no corpo**, não multipart — para não exigir
> `python-multipart` no backend.

### 3 · Revisão manual
Cada linha: checkbox · Data · Descrição · badge Débito/Crédito · Valor ·
Select de [[Tipo|tipo]].

Padrões inteligentes:
- **Créditos** (pagamentos/estornos) já vêm **desmarcados** — não são gastos
- **Tipo padrão = Conta** (1)
- **Já importadas** (FITID conhecido) vêm desmarcadas com badge "já importada"
- Há **"aplicar tipo às selecionadas"** para lotes
- **Busca por descrição** (mesmo `matchesSearch` das outras listas — ver
  [[ExpenseBox#Busca — é o ÚLTIMO elo da cadeia de filtros|ExpenseBox]])

> **Com a busca ativa, as ações em massa agem só sobre o que está visível.**
> "Selecionar todas" e "aplicar tipo às selecionadas" olham para `visibleRows`,
> não para `rows` — marcar em lote não pode mexer no que o usuário não está
> vendo. Já o **commit continua enviando todas as selecionadas**, inclusive as
> escondidas pela busca (é a verdade do que vai ser gravado); por isso o rodapé
> avisa "N fora da busca" quando isso acontece.

### 4 · Vencimento e commit
O usuário informa o **vencimento da fatura** via [[DatePicker-e-Select|DatePicker]].
`importCommit(items)` grava cada transação como lançamento **`UNICA`** no mês do
vencimento, com `na_fatura = true` e `data_compra` = data original.

Resposta: `{ created, skipped }`.

---

## As duas datas

| | Vira | Papel |
|---|---|---|
| Data da transação no OFX | `data_compra` | Só informação |
| Vencimento informado | `data_inicio`/`data_fim` | **O mês do orçamento** |

Uma compra de 30/06 numa fatura de agosto é um lançamento **de agosto**.
Ver [[Fatura-de-Cartao]].

---

## Deduplicação por FITID

O **preview** marca `ja_importada = true` quando o FITID já existe.
O **commit** pula FITIDs já gravados — ou repetidos no mesmo lote.

**Dois furos:**
1. Transação **sem FITID** não é deduplicada — reimportar duplica.
2. Registros importados **antes da migração `0006_fitid`** não têm `fitid` →
   não são detectados. Limpeza manual.

---

## O que a tela NÃO faz

- **Não importa extrato de conta corrente** — o fluxo assume fatura de cartão
  (daí o campo de vencimento obrigatório).
- **Não aceita CSV, OFC ou PDF.** Só OFX.
- **Não categoriza automaticamente** — nenhuma [[Tag]] é aplicada, nem por
  padrão de descrição.
- **Não agrupa parcelamento.** Uma compra em 10x vira 10 lançamentos
  independentes, sem relação entre si.
- **Não tem staging.** Não há rascunho salvo: sair da tela perde a revisão.
- **Não edita valor ou descrição** na revisão — só tipo e inclusão.

---

## Testando

Arquivo de exemplo: `FinancialControllBackend/samples/exemplo_fatura.ofx`

---

## Relacionadas
[[Fatura-de-Cartao]] · [[Lancamento]] · [[Tipo]] · [[Loader]] · [[Endpoints]] · [[Migracoes]]
