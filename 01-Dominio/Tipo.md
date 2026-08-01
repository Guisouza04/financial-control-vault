---
aliases: ["Tipo", "Bucket", "60/20/10/10", "Divisão do salário"]
tags: [dominio]
codigo: ["FinancialControll2.0/src/pages/Home/index.jsx"]
atualizado: 2026-08-01
---

# Tipo (os quatro baldes)

O `tipo` é **o mecanismo central de roteamento** do sistema: ele conecta página,
endpoint, componente e fatia do orçamento. É o *plano* — quanto do salário
deveria ir para onde.

| tipo | Balde | % do salário | Natureza do limite | Rota | Endpoint |
|---|---|---|---|---|---|
| 1 | **Contas** | 60% | `teto` — não pode passar | `/Contas` | `GET /financas/contas` |
| 2 | **Investimentos** | 20% | `meta` — alvo a atingir | `/Investments` | `GET /financas/investimentos` |
| 3 | **Opcionais** | 10% | `teto` — não pode passar | `/Optional` | `GET /financas/opcionais` |
| 4 | **Metas** | 10% | `meta` — alvo a atingir | `/metas` | `GET /financas/metas` |

---

## Teto vs. Meta — a distinção que muda a cor

Esta é a sutileza que o Dashboard implementa e que é fácil quebrar sem perceber:

- **`teto`** (Contas, Opcionais): gastar **menos** é bom. Passar do limite fica
  **vermelho**.
- **`meta`** (Investimentos, Metas): guardar **mais** é bom. Atingir o alvo fica
  **verde**.

Ou seja: **80% de um teto é tranquilizador; 80% de uma meta é um alerta.**
A mesma barra em 80% significa coisas opostas dependendo do bucket. Se você
mexer nos medidores, essa é a regra a preservar.

Configuração em `BUDGET`, `src/pages/Home/index.jsx` — cada bucket tem `pct` e
`kind`.

---

## Paleta

Cores fixas por bucket, validadas para contraste e daltonismo sobre o fundo
roxo (`--background: #1E0033`):

| Bucket | Cor |
|---|---|
| Contas | `#3987e5` |
| Investimentos | `#199e70` |
| Opcionais | `#c98500` |
| Metas | `#d55181` |

> **Cores de status (verde/vermelho de pago/pendente) nunca são reutilizadas
> como cor de série.** Se fossem, um bucket ficaria "verde" por acaso e leria
> como "está tudo certo".

---

## Metas são o tipo 4, mas não se comportam como os outros

Tipos 1–3 usam o mesmo componente ([[ExpenseBox]]) com uma tabela. O tipo 4 tem
tela própria com cards de progresso, porque uma meta é um **aporte acumulativo**
e não um boleto. Ver [[Meta]].

Consequência prática: **metas não são criáveis pelo [[ExpenseBox]] nem pelo
[[QuickAddModal]]** — a constante `TIPO_OPTIONS` exclui o 4 deliberadamente.
Criar meta é ação da própria página `/metas`.

---

## Limites deste modelo

- **Os percentuais são fixos no código.** O usuário não configura a divisão —
  60/20/10/10 é hardcoded em `BUDGET`. Deixar isso configurável é uma mudança
  de produto, não só de código (afeta o significado do dashboard inteiro).
- **Não há tipo 5.** Adicionar um bucket exige tocar: `BUDGET`, o mapa de
  endpoints, o Nav, as rotas e a soma de percentuais (que precisa fechar 100%).
- **Não há subtipo/hierarquia.** Profundidade de categorização é papel da [[Tag]].

---

## Relacionadas
[[Lancamento]] · [[Meta]] · [[Tag]] · [[Salario]] · [[Tela-Dashboard]]
