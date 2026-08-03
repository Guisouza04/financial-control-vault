---
aliases: ["Settings", "Configurações"]
tags: [tela]
rota: "/Settings"
codigo: ["FinancialControll2.0/src/pages/Settings/index.jsx"]
atualizado: 2026-08-01
---

# Tela · Configurações

**Rota:** `/Settings` · **Componente:** `src/pages/Settings` (exporta `Config`)

**Tela de nível raiz** — é o 4º item do [[Nav]]. Absorveu a antiga
[[Tela-Dados|tela de Dados]]: a hierarquia estava invertida (o menu levava a
"Dados", e era *lá dentro* que havia um card para "Configurações"), quando
"Dados" é uma configuração como qualquer outra. Hoje é uma tela só, com quatro
cards e quatro modais.

| Card | Ação |
|---|---|
| **Perfil** | Modal — nome, apelido, e-mail → `PUT /security/profile` |
| **Dados** | Modal de **Salário** → `POST /financas/salary` |
| **Alterar Senha** | Modal → `PUT /security/change-password` |
| **Sair** | Modal de confirmação → logout |

Não há botão "Voltar": a tela não tem mais pai.

> **`/dados` continua existindo como redirect** (`<Navigate to="/Settings" replace />`
> no `App.jsx`). Links antigos apontando para lá não quebram.

---

## Modal de Salário — a configuração mais importante do sistema

É onde nasce o denominador de todo o [[Tela-Dashboard|Dashboard]]. Ver [[Salario]].

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

> ✅ Endpoint confirmado no backend (`app/api/security.py:69`).

---

## Alterar senha

Três campos: senha antiga, nova, confirmar nova.

**Validações client-side:**
- Todos preenchidos
- Nova senha === confirmação

Payload: `{ oldPassword, newPassword }` — **camelCase**, diferente do
`snake_case` do resto da API de finanças. Ver [[Endpoints]].

---

## Logout

```js
localStorage.removeItem('authToken');
navigate("/Login");
```

**O logout é puramente client-side.** Não há chamada ao backend, não há
invalidação de token no servidor. Um JWT vazado continua válido até expirar.
Ver [[Autenticacao]].

---

## Comportamentos

- Os modais de Perfil e Salário **não carregam os valores atuais** — abrem
  sempre vazios. Não é edição de um estado carregado, é envio de um novo valor.
  Ver [[Pendencias]].
- Cancelar limpa o formulário.
- Feedback via [[Toast]].
- Os quatro modais são artesanais (`useState` + `.modalOverlay`), não usam o
  [[ConfirmDialog]] global — inclusive o de logout.

---

## O que a tela NÃO faz

- **Não mostra o salário nem o perfil atuais** — os modais abrem vazios, sem
  `GET`.
- **Não tem "esqueci minha senha"** — só troca com a senha antiga em mãos.
- **Não tem 2FA**, sessões ativas, ou revogação de token.
- **Não permite excluir a conta.**
- **Não tem preferências** (tema, idioma, moeda, notificações).
- **Não tem histórico de salário.**

---

## Relacionadas
[[Salario]] · [[Card]] · [[Nav]] · [[Autenticacao]] · [[Tela-Dashboard]] · [[Endpoints]]
