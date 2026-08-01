---
aliases: ["TagPicker"]
tags: [componente]
codigo: ["FinancialControll2.0/src/components/TagPicker/index.jsx", "FinancialControll2.0/src/hooks/useTags.js"]
atualizado: 2026-08-01
---

# TagPicker

**`src/components/TagPicker`**

Seleção múltipla de [[Tag|tags]], com criação inline e exclusão global.
Usado no modal do [[ExpenseBox]] e do [[QuickAddModal]].

**Não** é usado no modal de [[Meta|meta]] — decisão de produto.

---

## O que faz

| Ação | Efeito |
|---|---|
| Selecionar/desselecionar | Monta o array `tag_ids` do lançamento |
| Criar inline | `POST /financas/tags` com nome + cor da paleta |
| Excluir | `DELETE /financas/tags/{id}` — **global**, com confirmação |

Estado vem de `useTags()` (busca 1×, expõe `addTag`/`editTag`/`removeTag`).

---

## Duas armadilhas

### 1 · `tag_ids` substitui, não faz merge

O array enviado é o **conjunto completo**. O backend troca a associação
inteira. Para adicionar uma tag, mande as antigas **mais** a nova.
Omitir o campo (default `[]`) **remove todas**. Ver [[Tag]].

### 2 · Excluir tag é global e irreversível

Some das ligações de **todos** os lançamentos — não só do que está aberto no
modal. Os lançamentos permanecem, apenas sem aquela tag.

Por isso a exclusão passa por [[ConfirmDialog]]. É a única ação do modal cujo
efeito escapa do lançamento sendo editado.

---

## Cor vem de paleta fixa

`src/utils/tagColors.js`. **Não é color picker livre** — garante contraste sobre
o fundo roxo e evita duas tags visualmente idênticas.

---

## Regras do backend a lembrar

- Nome **único por usuário**, case-insensitive (`uq_tag_user_nome`)
- Nome ≤ 40 caracteres
- IDs de tag de **outro usuário** são silenciosamente ignorados no
  create/update de lançamento

---

## Relacionadas
[[Tag]] · [[ExpenseBox]] · [[QuickAddModal]] · [[ConfirmDialog]]
