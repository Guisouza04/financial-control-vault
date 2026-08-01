---
aliases: ["O que o sistema não faz", "Limites", "Fora de escopo"]
tags: [limite]
atualizado: 2026-08-01
---

# O que o sistema NÃO faz

A nota mais útil para decidir o que construir. Separa **limite deliberado**
(faz parte da identidade do produto) de **lacuna** (ainda não foi feito).

> **Ao pedir uma feature nova, procure aqui primeiro.** Se está na coluna
> "deliberado", a conversa é sobre mudar o produto, não sobre implementar.

---

## Modelo de dados

| Não faz | Natureza |
|---|---|
| Valor diferente por parcela (conta de luz que varia) | 🔒 Deliberado — quebraria `alvo = valor × ocorrências`. Alternativa: lançamentos `UNICA` por mês |
| Observação, anexo ou comprovante no lançamento | 🕳️ Lacuna |
| Histórico de alteração / auditoria | 🕳️ Lacuna |
| Múltiplas moedas | 🔒 Deliberado — tudo é BRL implícito |
| Hierarquia de [[Tag\|tags]] | 🔒 Deliberado |
| Tag por parcela (só por lançamento) | 🔒 Deliberado — ver [[Tag]] |
| Bucket além dos 4, ou percentuais configuráveis | 🔒 Deliberado — 60/20/10/10 é a tese do produto |

## Recorrência

| Não faz | Natureza |
|---|---|
| Quinzenal, semanal, diária | 🕳️ Lacuna (quinzenal já foi cogitada) |
| Dia útil ("dia 5 ou próximo útil") | 🕳️ Lacuna |
| Recorrência sem fim | 🔒 Deliberado — todo recorrente tem `data_fim` |
| Pular uma ocorrência | 🕳️ Lacuna |
| `ANUAL` além de 5 anos | 🔒 Deliberado (`MAX_ANUAL_YEARS`) |

## Salário e orçamento

| Não faz | Natureza |
|---|---|
| Histórico de salário | 🕳️ Lacuna — **consequente**: o Dashboard de março usa o salário de hoje |
| Múltiplas fontes de renda, 13º, freela | 🕳️ Lacuna |
| Orçamento por ciclo de pagamento (dia 5 a dia 5) | 🔒 Por ora — competência é mês calendário. `periodo_pagamento` seria o gancho. Ver [[Salario]] |
| Saldo bancário real | 🔒 Deliberado — o sistema não conhece conta bancária |

## Cartão

| Não faz | Natureza |
|---|---|
| Entidade "fatura" (fechar, agrupar) | 🔒 Deliberado — ver [[ADR-001-naFatura-e-marcador]] |
| Cadastro de cartões, limite, bandeira | 🕳️ Lacuna |
| Cálculo automático do vencimento | 🕳️ Lacuna — o usuário informa a cada importação |
| Relacionar parcelas de uma compra em 10x | 🕳️ Lacuna |
| Proteger contra lançar a fatura **e** as compras | 🕳️ Lacuna conhecida |

## Importação

| Não faz | Natureza |
|---|---|
| CSV, OFC, PDF | 🕳️ Lacuna — só OFX |
| Extrato de conta corrente | 🕳️ Lacuna — o fluxo assume fatura de cartão |
| Categorização automática por descrição | 🕳️ Lacuna — **candidata forte a próxima feature** |
| Staging / rascunho da revisão | 🔒 Deliberado — stateless. Sair perde a revisão |
| Dedup de transação sem FITID | 🕳️ Furo conhecido — ver [[ADR-007-Dedup-por-FITID]] |

## Análise

| Não faz | Natureza |
|---|---|
| Comparar meses / série temporal / tendência | 🕳️ Lacuna — **a maior**, dado que o app tem 1 ano de histórico potencial |
| Projeção de saldo futuro | 🕳️ Lacuna |
| Exportar (CSV, PDF, relatório) | 🕳️ Lacuna |
| Busca por texto nos lançamentos | 🕳️ Lacuna |
| Ordenar a tabela por coluna | 🕳️ Lacuna |
| Ações em massa (excluir/editar vários) | 🕳️ Lacuna |

## Conta e segurança

| Não faz | Natureza |
|---|---|
| Recuperar senha ("esqueci minha senha") | 🕳️ Lacuna — **nem endpoint, nem tela** |
| Login social (os botões são placeholder) | 🕳️ Lacuna |
| Refresh token / revogação / logout remoto | 🕳️ Lacuna — ver [[Autenticacao]] |
| 2FA | 🕳️ Lacuna |
| Confirmação de e-mail no cadastro | 🕳️ Lacuna |
| Rate limit de login | 🕳️ Lacuna |
| Excluir a própria conta | 🕳️ Lacuna |
| Multiusuário / família / compartilhamento | 🔒 Deliberado — é um app pessoal |

## Operação

| Não faz | Natureza |
|---|---|
| Deploy fora do localhost | 🕳️ `baseURL` **hardcoded** em `api.js` — sem env var |
| Testes automatizados | 🕳️ Lacuna — só ESLint |
| Notificações / lembrete de vencimento | 🕳️ Lacuna |
| PWA / app mobile | 🕳️ Lacuna (a UI é responsiva) |
| Backup / exportação de dados | 🕳️ Lacuna |

---

## Se eu fosse escolher as próximas três

Opinião, não decisão — vale discutir:

1. **Comparação entre meses** no [[Tela-Dashboard|Dashboard]]. O dado já existe
   e é o que transforma o app de "registro" em "análise".
2. **Categorização automática na importação** (regra: descrição → [[Tag]]).
   Ataca o maior atrito do fluxo mais trabalhoso.
3. **`VITE_API_URL`** em vez de `baseURL` hardcoded. Mudança de 10 minutos que
   desbloqueia qualquer deploy.

---

## Relacionadas
[[Pendencias]] · [[Mapa-do-Sistema]] · [[05-Decisoes/ADR-Index|ADRs]]
