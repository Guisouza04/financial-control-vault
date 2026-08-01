---
aliases: ["Login", "Cadastro", "TelaLogin"]
tags: [tela]
rota: "/Login"
codigo: ["FinancialControll2.0/src/pages/Login/index.jsx", "FinancialControll2.0/src/components/LoginForm/index.jsx"]
atualizado: 2026-08-01
---

# Tela · Login

**Rota:** `/Login` · **A única rota pública** — todas as outras são envolvidas
por `PrivateRoute`.

Alterna entre dois formulários: `LoginForm` e `SignInForm` (cadastro).

---

## Login

`POST /security/login`

O campo `username` aceita **CPF ou e-mail** — o backend resolve os dois.
É convenção do projeto, registrada no `CLAUDE.md` do backend.

**Resposta:**
```json
{ "success": true, "authToken": "JWT" }
```

O token vai para `localStorage.authToken`. A partir daí, o interceptor do
[[Camada-de-Dados|api.js]] injeta `Authorization: Bearer {token}` em **todas**
as requisições.

---

## Cadastro

`POST /security/signup` com `{ CPF, email, password, confirmPassword }`.

> Note o `CPF` **maiúsculo** no payload — inconsistente com o resto, mas é o
> contrato atual.

---

## Botões de login social são placeholder

Google, Apple e Microsoft (`src/assets/IconGoogle.svg` etc.) estão na tela e
**não fazem nada**. Não há OAuth configurado, nem no front nem no back.

Ver [[Pendencias]].

---

## O que a tela NÃO faz

- **Não tem "esqueci minha senha"** — não há fluxo de recuperação em lugar
  nenhum do sistema.
- **Não valida CPF** (dígito verificador) no front.
- **Não tem confirmação de e-mail.**
- **Não tem rate limit** visível de tentativas.

---

## Armadilha de sessão

`PrivateRoute` só checa **se existe** `localStorage.authToken` — não valida
assinatura nem expiração. Com um token expirado, a navegação passa e as
chamadas de API é que falham (401). Ver [[Autenticacao]].

---

## Relacionadas
[[Autenticacao]] · [[Camada-de-Dados]] · [[Tela-Settings]]
