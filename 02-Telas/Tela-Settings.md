---
aliases: ["Settings", "Configurações"]
tags: [tela]
rota: "/Settings"
codigo: ["FinancialControll2.0/src/pages/Settings/index.jsx"]
atualizado: 2026-08-01
---

# Tela · Configurações

**Rota:** `/Settings` · **Componente:** `src/pages/Settings` (exporta `Config`)

Subtela de [[Tela-Dados|Dados]] — o título é literalmente
`"Dados > Configurações"`. Dois cards.

| Card | Ação |
|---|---|
| **Alterar Senha** | Modal → `PUT /security/change-password` |
| **Sair** | Modal de confirmação → logout |

Mais um botão "Voltar" para `/Dados`.

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

> **Isto já limpa o token.** O `CLAUDE.md` menciona em um ponto que o logout
> "não limpa o localStorage (bug conhecido)" e em outro que foi resolvido — a
> primeira menção está obsoleta. O código atual (`Settings/index.jsx:33`) limpa.
> Ver [[Pendencias]].

**O logout é puramente client-side.** Não há chamada ao backend, não há
invalidação de token no servidor. Um JWT vazado continua válido até expirar.
Ver [[Autenticacao]].

---

## O que a tela NÃO faz

- **Não tem "esqueci minha senha"** — só troca com a senha antiga em mãos.
- **Não tem 2FA**, sessões ativas, ou revogação de token.
- **Não permite excluir a conta.**
- **Não tem preferências** (tema, idioma, moeda, notificações).

---

## Relacionadas
[[Tela-Dados]] · [[Autenticacao]] · [[Endpoints]]
