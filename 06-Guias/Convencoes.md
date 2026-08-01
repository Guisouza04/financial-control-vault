---
aliases: ["Convenções", "Estilo"]
tags: [guia]
atualizado: 2026-08-01
---

# Convenções do Projeto

---

## Estrutura de componente

```
ComponentName/
├── index.jsx      ← lógica + JSX
└── styles.js      ← styled-components
```

Sem exceção. Estilo **nunca** fica no `index.jsx`.

---

## Idioma

| O quê | Idioma |
|---|---|
| Features, páginas, rotas, labels | **Português** (`Despesas`, `Metas`, `de_conta`) |
| Lógica técnica, helpers, hooks | **Inglês** (`useAccounts`, `transformAccount`) |

Mistura-se no mesmo arquivo, e tudo bem — a fronteira é *domínio* vs. *técnica*.

> Consequência visível: o backend fala `snake_case` **em português**
> (`vl_conta`, `dia_vencimento`), o frontend fala `camelCase` **em inglês**
> (`value`, `dueDay`). A tradução vive só em `transformAccount`.
> Ver [[Camada-de-Dados]].

---

## Nomenclatura de endpoints

- Financeiro → `/financas/*`
- Autenticação → `/security/*`
- Tudo sob o prefixo `/api`

> `/security/*` usa `camelCase` no payload (`confirmPassword`, `oldPassword`),
> `/financas/*` usa `snake_case`. Inconsistente, mas é o contrato. Ver
> [[Endpoints]].

---

## Estado

**Não há estado global de dados.** Sem Redux, sem Context para dados de
negócio. Cada página tem sua instância de `useAccounts`.

Context é usado **só** para UI transversal: [[Toast]] e [[ConfirmDialog]].

> **Context e hook em arquivo separado** do componente (`src/context/toast.js`,
> `src/context/confirm.js`). Exportar hook e componente do mesmo arquivo quebra
> o **Fast Refresh** do Vite.

---

## Controles nativos não estilizáveis são substituídos

`<select>` → [[DatePicker-e-Select|`Select`]]
`<input type="date">` → [[DatePicker-e-Select|`DatePicker`]]

Contrato: gatilho + popup, fecham no clique fora e no Esc, **`onChange` recebe
o valor, não um evento**. Ver [[ADR-004-Controles-customizados]].

---

## Retorno dos métodos de dados

Nunca lançam. Sempre:

```js
{ success: true, data? }
{ success: false, error: "mensagem" }
```

A UI decide o [[Toast]]. Ver [[Camada-de-Dados]].

---

## Estilo global

`src/styles/globalStyles.js`

```css
--background:      #1E0033   /* roxo escuro — fundo principal */
--RoxoNubank:      #820AD1   /* roxo primário */
--Complementar:    #E5CCFF   /* roxo claro */
--RoxoClaro:       #A45DE7
--RoxoEscuro:      #4F0186
--TextSecundarios: #B3B3B3
```

**Classes utilitárias:**

| Classe | |
|---|---|
| `.frame` | Grid `auto 1fr` — sidebar [[Nav]] + conteúdo |
| `.containerExpenses` | Flex column, padding 5rem, gap 4rem |
| `.button2` / `.button3` | Botão roxo gradiente / branco |
| `.modalOverlay` / `.defaultModal` | Overlay fixo + modal centralizado |

**Base:** `font-size: 62.5%` no `html` → **`1rem = 10px`**

> Toda medida em `rem` no projeto assume isso. `1.6rem` = 16px.

---

## Cores de dados

**Paleta categórica por [[Tipo|bucket]]** (validada para contraste/CVD sobre o
roxo): Contas `#3987e5` · Investimentos `#199e70` · Opcionais `#c98500` ·
Metas `#d55181`

**Cores de status:** `good #3ddc84` · `warning #fab219` · `critical #ff6b6b`

> ⚠️ **Cor de status nunca é reutilizada como cor de série.** Senão um bucket
> fica verde por acaso e lê como aprovação.

---

## Comentários

O padrão do projeto é **comentar o porquê, não o quê**. Os melhores comentários
do código (em `recurrence.js`, `pagamento.py`, `lancamento.py`) explicam a
decisão e o bug que a motivou.

Mantenha isso — é o que faz o código sobreviver sem este vault ao lado.

---

## Relacionadas
[[Camada-de-Dados]] · [[Ambiente-de-Dev]] · [[Guia-Antes-de-Implementar]] · [[ADR-004-Controles-customizados]]
