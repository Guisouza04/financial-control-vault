---
aliases: ["Tag", "Tags", "Categorização"]
tags: [dominio]
codigo: ["FinancialControllBackend/app/models/tag.py", "FinancialControll2.0/src/hooks/useTags.js"]
atualizado: 2026-08-01
---

# Tag

Rótulo livre do usuário ("Mercado", "Combustível", "Pet") para categorizar
[[Lancamento|lançamentos]]. Existe porque o [[Tipo]] responde uma pergunta de
**orçamento** e a tag responde uma de **natureza**:

| Pergunta | Responde |
|---|---|
| "De qual fatia do salário isso sai?" | [[Tipo]] — o plano 60/20/10/10 |
| "O que é isso, afinal?" | **Tag** |

Por isso a tag é **transversal ao tipo**: `Mercado` pode aparecer em Contas
(a compra do mês) e em Opcionais (o supérfluo do fim de semana).

---

## Modelo

**`tags`** — `id`, `user_id`, `nome` (≤40), `cor` (hex `#RRGGBB`), `dt_create`
Com `UniqueConstraint(user_id, nome)` — nome único **por usuário**,
case-insensitive.

**`lancamento_tags`** — associação N:N, `CASCADE` nos dois lados.

Migração: `0007_tags`.

---

## A decisão: tag é do LANÇAMENTO, não da parcela

Uma tag vale para **todas as ocorrências** de um recorrente. Não existe
"em agosto essa conta foi Mercado, em setembro foi Lazer".

Isso é deliberado e é o que separa tag de [[Competencia-e-Pagamento|pagamento]]:

| | Granularidade | Por quê |
|---|---|---|
| `pagamentos` | **Por competência** | Cada mês é pago independentemente — é da natureza do problema |
| `tags` | **Por lançamento** | A natureza do gasto não muda mês a mês; granularidade por parcela seria complexidade sem demanda |

Se um dia surgir necessidade real de tag por parcela, isso é uma mudança de
modelo — não um ajuste.

---

## `tag_ids` substitui, não faz merge

No create/update de lançamento, `tag_ids` é o **conjunto completo** de tags:

```json
{ "de_conta": "...", "tag_ids": [3, 7] }
```

O backend **substitui** a associação inteira. Consequências:

- Omitir `tag_ids` (default `[]`) **remove todas as tags** do lançamento.
- Para adicionar uma tag, envie as antigas **mais** a nova.
- IDs de tag de **outro usuário** são silenciosamente ignorados.

> É por isso que o modal de [[Meta|meta]] não tem [[TagPicker]] e ainda assim
> a meta fica sem tags — omitir o campo já produz o resultado esperado.

---

## Excluir tag é global

Deletar uma tag (`DELETE /financas/tags/{id}`) remove as **ligações** com todos
os lançamentos, mas **não apaga nenhum lançamento**. Eles apenas ficam sem
aquela tag. A UI ([[TagPicker]]) pede confirmação porque a ação afeta tudo de
uma vez, não só o lançamento aberto.

---

## Cor

Escolhida de uma paleta fixa (`src/utils/tagColors.js`) — não é color picker
livre. Motivo: garantir contraste legível sobre o fundo roxo e evitar que o
usuário crie duas tags visualmente indistinguíveis.

---

## Onde as tags aparecem

- **[[ExpenseBox]]** — chips coloridos na coluna Nome + filtro por tag
- **Modal de criar/editar** — [[TagPicker]]
- **[[Tela-Dashboard]]** — seção "Gastos por tag", somando **todos** os tipos

### O detalhe do "Gastos por tag"

Como um lançamento pode ter **várias** tags, o valor conta **em cada uma**.
A soma do ranking pode, portanto, **passar do total comprometido**. Isso é
esperado e está avisado no subtítulo da seção — não é bug.

Lançamentos sem tag caem no bucket neutro **"Sem tag"**.

---

## Limites

- **Sem hierarquia.** Não há tag-pai/tag-filha.
- **Sem tag automática.** Nada é tagueado por regra ou por padrão de descrição —
  nem na [[Tela-Importar-Extrato|importação de extrato]].
- **Sem cor livre.** Só a paleta.
- **Sem renomear em massa** ou merge de tags.

---

## Relacionadas
[[Tipo]] · [[Lancamento]] · [[TagPicker]] · [[Tela-Dashboard]]
