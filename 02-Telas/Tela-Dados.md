---
aliases: ["Dados", "Perfil"]
tags: [tela, historico]
rota: "/dados (redirect → /Settings)"
codigo: []
atualizado: 2026-08-01
---

# Tela · Dados — **absorvida por Configurações**

> ⚠️ **Esta tela não existe mais.** O conteúdo dela vive em
> [[Tela-Settings|Configurações]] (`/Settings`). A nota fica aqui porque
> `/dados` ainda responde (como redirect) e porque as decisões de produto sobre
> salário e perfil continuam valendo — só mudaram de endereço.

## O que aconteceu

A hierarquia estava invertida: o [[Nav]] levava a **"Dados"**, e era *dentro*
dessa tela que existia um card apontando para **"Configurações"**. Só que
"Dados" (salário) é uma configuração — não o contrário. Em 2026-08-01 as duas
telas viraram uma:

| Antes | Depois |
|---|---|
| Nav → "Dados" (`/dados`) | Nav → "Configurações" (`/Settings`) |
| `/dados`: Perfil · Configurações · Dados | `/Settings`: Perfil · Dados · Alterar Senha · Sair |
| `/Settings`: Alterar Senha · Sair (subtela, com "Voltar") | — (deixou de ser subtela) |

- `src/pages/Dados/` foi **removido**; `src/pages/Settings/` recebeu os estados,
  handlers e modais de Perfil e Salário como estavam.
- `/dados` virou `<Navigate to="/Settings" replace />` dentro do `PrivateRoute`
  — links antigos (como o aviso "Definir salário" do [[Tela-Dashboard|Dashboard]],
  que hoje já aponta direto para `/Settings`) não quebram.
- O ícone do Nav **não mudou**: já era uma engrenagem. Só a constante foi
  renomeada de `DadosIcon` para `ConfigIcon`.

---

## Relacionadas
[[Tela-Settings]] · [[Nav]] · [[Salario]] · [[Tela-Dashboard]]
