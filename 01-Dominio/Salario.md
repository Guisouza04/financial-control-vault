---
aliases: ["Salário", "Renda"]
tags: [dominio]
codigo: ["FinancialControllBackend/app/api/financas.py", "FinancialControll2.0/src/pages/Settings/index.jsx"]
atualizado: 2026-08-01
---

# Salário

A **base de cálculo de todo o [[Tela-Dashboard|Dashboard]]**. Sem ele, a divisão
60/20/10/10 não tem denominador: os medidores ficam sem limite e não existe
"saldo livre".

Mora em `users.salario` — **é campo do usuário, não um lançamento**. Não entra
em nenhuma listagem nem soma.

---

## Modelo

| Campo | Tipo | |
|---|---|---|
| `users.salario` | `Numeric(10,2)?` | **Nullable** — `NULL` = ainda não configurado |
| `users.periodo_pagamento` | `String(50)?` | Texto livre (ex.: "Dia 5") |

---

## Contrato

**`GET /financas/salary`** → `SalaryOut`
```json
{ "salario": "8000.00", "periodo_pagamento": "Dia 5" }
```
Ambos podem vir `null`. `salario` é serializado como **string** com 2 casas.

**`POST /financas/salary`** → `SalaryIn`
```json
{ "salario": 8000.00, "periodo_pagamento": "Dia 5" }
```

> ⚠️ **Os dois campos são obrigatórios no POST.** `SalaryIn` não tem default —
> enviar só `salario` resulta em **422**. Se um dia a UI permitir editar só o
> valor, o schema precisa mudar junto.

O front expõe via `financeService.fetchSalary()`, que normaliza para
`{ salario: number|null, periodoPagamento }`.

---

## `salario = NULL` é um estado de primeira classe

Não é erro — é o estado de quem acabou de criar a conta. O sistema **precisa
funcionar** sem salário:

- **Dashboard:** exibe aviso com link para `/Settings`. Os **medidores ficam sem
  limite** (não há teto a comparar), mas o **donut de distribuição continua
  funcionando** — ele mostra a proporção entre os buckets, que não depende do
  salário.
- **Telas de despesa:** indiferentes. Nunca consultam salário.

> Se você adicionar qualquer cálculo novo baseado em salário, **trate o `null`
> explicitamente**. Um `NaN` vazando para a UI é o sintoma clássico.

---

## `periodo_pagamento` é decorativo

Texto livre, guardado e exibido, mas **nenhuma lógica depende dele**. Não muda
competência, não desloca o mês do orçamento, não gera lembrete. É informação
para o usuário.

Se um dia o orçamento precisar rodar de "dia 5 a dia 5" em vez de mês
calendário, este campo vira o gancho — e aí [[Competencia-e-Pagamento]] muda
junto, porque competência é `"YYYY-MM"` (mês calendário) por definição.

---

## Limites

- **Um salário só, sem histórico.** Alterar sobrescreve. O Dashboard de março
  usa o salário de hoje, não o de março.
- **Sem outras fontes de renda.** Não há "renda extra", "13º" ou "freela".
- **Sem líquido vs. bruto.** É um número só.
- **Não é por competência.** Ver acima — essa é a limitação mais consequente,
  porque distorce a análise retroativa.

---

## Onde é editado

Modal do card **Dados** na [[Tela-Settings|tela de Configurações]] (`/Settings`).

---

## Relacionadas
[[Tipo]] · [[Tela-Dashboard]] · [[Tela-Settings]] · [[Endpoints]]
