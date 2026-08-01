---
aliases: ["Nav", "Menu de navegação", "MenuNavecacao"]
tags: [componente]
codigo: ["FinancialControll2.0/src/components/Nav/index.jsx"]
atualizado: 2026-08-01
---

# Nav (menu lateral)

**`src/components/Nav/index.jsx`** — exporta `MenuNavecacao`
*(o nome tem um typo histórico: "Navecacao". Está assim em todos os imports.)*

Montado manualmente por **cada página** dentro de `<div className="frame">`
(grid `auto 1fr` — sidebar + conteúdo). **Não é um layout compartilhado por
rota** — se você criar uma página nova, precisa montar o Nav nela.

---

## Os quatro itens

```js
{ to: "/",         label: "Dashboard" }
{ to: "/Financas", label: "Finanças"  }
{ to: "/metas",    label: "Metas"     }
{ to: "/dados",    label: "Dados"     }
```

> **[[Meta|Metas]] é item de primeiro nível**, não subitem de Finanças — porque
> é um dos quatro baldes do orçamento. Já Contas/Investimentos/Opcionais são
> alcançados pelo hub [[Tela-Financas|/Financas]].

---

## Dois modos, um botão

A setinha (`NavArrow`) faz coisas diferentes conforme a largura:

| Largura | Ação |
|---|---|
| **> 768px** (desktop) | Recolhe/expande a sidebar |
| **≤ 768px** (mobile) | Fecha o drawer |

Decidido em runtime via `window.matchMedia("(max-width: 768px)")`.

**No mobile** há ainda uma `MobileTopBar` com hambúrguer (☰) e `Backdrop`
clicável para fechar.

---

## O estado recolhido persiste

```js
localStorage.setItem("navCollapsed", String(next))
```

Lido na inicialização do `useState`. Sobrevive a navegação e a reload.

> É a **segunda** chave de `localStorage` do sistema, junto de `authToken`.
> Ver [[Autenticacao]].

---

## Sobre a caixa das rotas

O Nav aponta para `/Financas` (maiúsculo), `/metas` e `/dados` (minúsculos), e
os cards de [[Tela-Financas]] apontam para `/contas`, `/investments`,
`/optional` — enquanto o `App.jsx` registra `/Contas`, `/Investments`,
`/Optional`.

**Isso funciona porque o React Router 7 casa rotas de forma
case-insensitive por padrão.** Ver [[ADR-006-Caixa-das-rotas]] — há uma nota
obsoleta no `CLAUDE.md` afirmando o contrário.

---

## Relacionadas
[[Tela-Financas]] · [[Tela-Dashboard]] · [[ADR-006-Caixa-das-rotas]] · [[Convencoes]]
