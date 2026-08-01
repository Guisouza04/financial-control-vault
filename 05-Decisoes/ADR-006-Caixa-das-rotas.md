---
aliases: ["ADR-006", "Caixa das rotas", "Case das rotas"]
tags: [adr]
atualizado: 2026-08-01
---

# ADR-006 · A caixa das rotas é inconsistente — e isso não quebra nada

## Contexto

As rotas do `App.jsx` misturam caixas, e **os links não batem com elas**:

| Registrado em `App.jsx` | Quem aponta | Como aponta |
|---|---|---|
| `/Contas` | [[Tela-Financas]] | `/contas` |
| `/Investments` | [[Tela-Financas]] | `/investments` |
| `/Optional` | [[Tela-Financas]] | `/optional` |
| `/Settings` | [[Tela-Dados]] | `/settings` |
| `/dados` | [[Tela-Settings]] | `/Dados` |
| `/metas` | [[Nav]] | `/metas` ✅ |
| `/Financas` | [[Nav]] | `/Financas` ✅ |

## O que o `CLAUDE.md` diz — e por que está errado

> *"Case-sensitive: … O React Router diferencia caixa — a rota precisa
> continuar exatamente `/metas`."*

**Isso não procede.** O React Router (v6+, e o projeto usa **7.8**) casa rotas
de forma **case-insensitive por padrão** — `caseSensitive` é uma opção por rota,
default `false`, e nenhuma rota do projeto a define.

A evidência mais direta é o próprio app: se o matching fosse sensível à caixa,
**três dos quatro cards do hub de Finanças cairiam no 404**, porque apontam
para `/contas`, `/investments` e `/optional` enquanto as rotas são
capitalizadas. O hub estaria visivelmente quebrado.

## Decisão

**A inconsistência é inofensiva e fica como está.** Nenhuma rota precisa "ser
exatamente" uma caixa específica.

O que muda é a **documentação**: a nota de case-sensitivity do `CLAUDE.md` é
incorreta e induz a cuidado desnecessário (e a conclusões erradas sobre onde
está um bug). Ver [[Pendencias]].

## Alternativas

| Alternativa | Avaliação |
|---|---|
| Padronizar tudo em minúsculo | Cosmético; mexe em ~8 arquivos sem ganho funcional. Vale se for junto de outro refactor de rotas |
| Marcar `caseSensitive: true` e alinhar | Adiciona rigidez sem benefício para um app pessoal |
| **Deixar como está e corrigir a doc** ✅ | Custo zero, remove a informação errada |

## Consequências

**A preservar** ⚠️
> **Não "conserte" um link por achar que a caixa está causando 404.** Se uma
> rota der 404, a causa é outra — caminho inexistente, `PrivateRoute`
> redirecionando por falta de token, ou erro de digitação de verdade.

**Se um dia padronizar:** faça em um único passo, tocando `App.jsx`, [[Nav]],
[[Tela-Financas]], [[Tela-Dados]] e [[Tela-Settings]] juntos — e atualize esta
nota.

---

## Relacionadas
[[Nav]] · [[Convencoes]] · [[Pendencias]]
