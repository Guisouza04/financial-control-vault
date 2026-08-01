---
aliases: ["Dados", "Perfil"]
tags: [tela]
rota: "/dados"
codigo: ["FinancialControll2.0/src/pages/Dados/index.jsx"]
atualizado: 2026-08-01
---

# Tela · Dados

**Rota:** `/dados` (registrada em **minúsculo** no `App.jsx`) ·
**Componente:** `src/pages/Dados`

Hub de configuração do usuário. Três cards, sendo **dois deles modais na própria
tela** e um link para [[Tela-Settings|Configurações]].

| Card | Abre |
|---|---|
| **Perfil** | Modal — nome, apelido, e-mail → `PUT /security/profile` |
| **Configurações** | Navega para [[Tela-Settings\|/Settings]] |
| **Dados** | Modal de **Salário** → `POST /financas/salary` |

---

## Modal de Salário — a configuração mais importante do sistema

É onde nasce o denominador de todo o [[Tela-Dashboard|Dashboard]].
Ver [[Salario]].

**Campos:**
- **Salário** — máscara BRL (`formatDigitsAsBRL`), guardado como string de
  dígitos (centavos) e convertido com `digitsToApiValue` → `"3500.00"`
- **Período de pagamento** — [[DatePicker-e-Select|Select]] com quatro opções:
  `Todo 5º dia útil` (default) · `Quinzenal` · `Todo dia 5` · `Personalizado`
- **Campo livre** aparece **só** quando "Personalizado" está selecionado

**Validações (client-side):**
- Salário precisa ser positivo (`hasPositiveValue`)
- "Personalizado" exige o texto livre preenchido

> **O backend exige os dois campos.** `SalaryIn` não tem default — mandar só
> `salario` dá **422**. Ver [[Endpoints]].

> **`periodo_pagamento` é decorativo.** Nenhuma lógica depende dele — não muda
> competência nem desloca o mês do orçamento. Ver [[Salario]].

---

## Modal de Perfil

`PUT /security/profile` com `{ nome, apelido, email }`.
**Nome e e-mail obrigatórios**; apelido é opcional.

> ✅ **Endpoint confirmado no backend** (`app/api/security.py:69`). O
> `CLAUDE.md` ainda lista isso como "validar se o backend implementa" — está
> desatualizado. Ver [[Pendencias]].

---

## Comportamentos

- Os modais **não carregam os valores atuais** — abrem sempre vazios. Não é
  edição de um estado carregado, é envio de um novo valor.
- Cancelar limpa o formulário.
- Feedback via [[Toast]].

---

## O que a tela NÃO faz

- **Não mostra o salário atual** — o modal abre vazio, sem `GET /financas/salary`.
- **Não mostra o perfil atual** — idem.
- **Não permite excluir a conta.**
- **Não tem histórico de salário.**

---

## Relacionadas
[[Salario]] · [[Tela-Settings]] · [[Tela-Dashboard]] · [[Endpoints]]
