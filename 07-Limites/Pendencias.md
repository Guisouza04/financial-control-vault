---
aliases: ["Pendências", "Bugs conhecidos", "Dívida"]
tags: [limite]
atualizado: 2026-08-02
---

# Pendências e dívida conhecida

Diferente de [[O-que-o-sistema-nao-faz]] (escopo): aqui é o que **existe e está
torto**.

---

## Bugs / riscos reais

| # | Item | Onde | Gravidade |
|---|---|---|---|
| 1 | **`PUT /security/profile` não valida e-mail duplicado.** Confia na constraint → trocar para e-mail existente dá `IntegrityError` → **500** em vez de 400 amigável | `app/api/security.py:69` | 🟡 Média |
| 2 | **`baseURL` hardcoded** — sem `VITE_API_URL`. Impede qualquer deploy sem editar código | `src/services/api.js:4` | 🟡 Média |
| 3 | **`JWT_SECRET` default público** (`"troque-este-segredo-em-producao"`). Sem `.env`, sobe assim | `app/core/config.py:8` | 🟡 Média (🔴 se for a produção) |
| 4 | **Transação OFX sem FITID não é deduplicada** — reimportar duplica | [[ADR-007-Dedup-por-FITID]] | 🟢 Baixa |
| 5 | **Importações pré-migração `0006`** não têm `fitid` → não detectadas como duplicata. Limpeza manual | [[Migracoes]] | 🟢 Baixa |
| 6 | **Sem proteção contra dupla contagem** de fatura-lump + compras itemizadas | [[ADR-001-naFatura-e-marcador]] | 🟢 Baixa |
| 7 | **Modais de Perfil e Salário abrem vazios** — não carregam o valor atual. O usuário não vê o que está configurado | `src/pages/Settings/index.jsx` | 🟢 Baixa (UX) |
| 8 | **Login social é placeholder** — três botões que não fazem nada | [[Tela-Login]] | 🟢 Baixa |
| 9 | **O período não acompanha a troca de aba.** Sair de Contas/Agosto pela [[FinanceTabs]] cai em Investimentos/mês atual — o usuário refaz o filtro. Saída: levar `ano`/`mes` na URL e o [[ExpenseBox]] ler de lá | [[FinanceTabs]] | 🟢 Baixa (UX) |

---

## Correções a fazer no `CLAUDE.md` ⚠️

O `CLAUDE.md` do frontend tem **três informações desatualizadas ou incorretas**.
Como ele é carregado automaticamente em toda sessão, informação errada ali é
mais cara que em qualquer outro lugar.

| # | O que diz | Realidade |
|---|---|---|
| A | *"Case-sensitive: o React Router diferencia caixa"* | **Incorreto.** React Router 7 casa rotas case-insensitive por padrão. Ver [[ADR-006-Caixa-das-rotas]] |
| B | *"Logout: apenas navega para `/Login` — **não** limpa o localStorage (bug conhecido)"* | **Obsoleto.** `Settings/index.jsx:33` faz `localStorage.removeItem('authToken')`. O próprio arquivo se contradiz mais abaixo, em "Resolvidos recentemente" |
| C | *"Salário / Perfil — confirmar se o backend implementa"* | **Resolvido.** `POST /financas/salary` (`financas.py:347`) e `PUT /security/profile` (`security.py:69`) existem |

Também **faltam** no `CLAUDE.md`:
- O **interceptor de resposta 401** do `api.js` (limpa token e redireciona)
- O endpoint `PUT /security/change-password`
- O [[ConfirmDialog]] e o `useConfirm`
- O [[Nav]] colapsável com persistência em `localStorage.navCollapsed`

---

## Dívida estrutural aceita

Não são bugs — são custos conscientes. Não "conserte" sem ler o ADR.

| Dívida | Vem de | Custo recorrente |
|---|---|---|
| Dois caminhos em toda lógica de período (recorrência vs. legado) | Migração `0002` | Todo helper ramifica |
| `conta_paga` órfão, ainda gravado em alguns fluxos | Migração `0003` | Campo morto que parece vivo |
| Filtro client-side | [[ADR-005-Filtro-de-periodo-no-frontend]] | Não escala; 4 requests no Dashboard |
| Sem teste automatizado | — | Verificação 100% manual |
| `pages/Despesas` serve a rota `/Financas` | Renomeação incompleta | Confusão ao procurar o arquivo |
| `MenuNavecacao` (typo) | Histórico | Cosmético |

---

## Resolvidos (não reabrir)

- ✅ Logout limpa o token
- ✅ Perfil virou modal (hoje em [[Tela-Settings|/Settings]]), com submit real
- ✅ Backend parou de filtrar por `dt_create` (escondia recorrentes)
- ✅ Aporte órfão inflando progresso de meta → `goalCompetencias`
- ✅ Compras importadas sumindo do total → [[ADR-001-naFatura-e-marcador]]

---

## Relacionadas
[[O-que-o-sistema-nao-faz]] · [[Migracoes]] · [[05-Decisoes/ADR-Index|ADRs]] · [[Guia-Antes-de-Implementar]]
